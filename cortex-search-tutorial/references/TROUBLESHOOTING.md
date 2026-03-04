# Troubleshooting Cortex Search

## Common Errors and Solutions

### Error: "Insufficient privileges"

**Symptoms:**
```
SQL access control error: Insufficient privileges to operate on schema
```
or
```
Cannot create Cortex Search Service
```

**Causes & Solutions:**

1. **Missing CORTEX_USER role**
   ```sql
   -- The role creating the service needs SNOWFLAKE.CORTEX_USER
   GRANT DATABASE ROLE SNOWFLAKE.CORTEX_USER TO ROLE my_role;
   ```

2. **Missing CREATE privilege on schema**
   ```sql
   GRANT CREATE CORTEX SEARCH SERVICE ON SCHEMA my_schema TO ROLE my_role;
   ```

3. **Missing SELECT on source table**
   ```sql
   GRANT SELECT ON TABLE support_transcripts TO ROLE my_role;
   ```

4. **Missing USAGE on warehouse**
   ```sql
   GRANT USAGE ON WAREHOUSE my_wh TO ROLE my_role;
   ```

### Error: "Service creation failed" or timed out

**Symptoms:**
- CREATE CORTEX SEARCH SERVICE hangs or times out
- Service shows error state after creation

**Causes & Solutions:**

1. **Warehouse too small for dataset**
   ```sql
   -- Use a larger warehouse for initial index build
   ALTER WAREHOUSE my_wh SET WAREHOUSE_SIZE = 'MEDIUM';
   
   -- After creation, you can scale back down
   ALTER WAREHOUSE my_wh SET WAREHOUSE_SIZE = 'XSMALL';
   ```

2. **Source query returns too much data**
   ```sql
   -- Test the source query first
   SELECT COUNT(*) FROM my_source_table;
   
   -- For very large tables, consider filtering
   CREATE CORTEX SEARCH SERVICE my_service
     ON text_col
     ATTRIBUTES region
     WAREHOUSE = my_wh
     TARGET_LAG = '1 hour'
     AS (
       SELECT * FROM my_table
       WHERE created_date > '2024-01-01'  -- Limit data scope
     );
   ```

3. **Source query not compatible with incremental refresh**
   
   The source query must be compatible with dynamic table incremental refresh. Avoid:
   - Non-deterministic functions (e.g., `RANDOM()`, `CURRENT_TIMESTAMP()`)
   - Certain types of JOINs or subqueries
   
   See [Snowflake docs on supported queries for dynamic tables](https://docs.snowflake.com/en/user-guide/dynamic-tables-tasks-create#supported-queries).

### Error: "Column not found" or "Invalid column"

**Symptoms:**
```
Column 'agent_id' is not a filterable attribute
```

**Causes & Solutions:**

1. **Filtering on a non-ATTRIBUTES column**
   
   Only columns listed in the `ATTRIBUTES` clause can be filtered:
   ```sql
   -- agent_id is NOT in ATTRIBUTES, so it can't be filtered
   CREATE CORTEX SEARCH SERVICE my_service
     ON transcript_text
     ATTRIBUTES region, product_category  -- Only these are filterable
     ...
   ```
   
   **Fix**: Recreate the service with the column in ATTRIBUTES:
   ```sql
   CREATE OR REPLACE CORTEX SEARCH SERVICE my_service
     ON transcript_text
     ATTRIBUTES region, product_category, agent_id  -- Added agent_id
     ...
   ```

2. **Column not in source query**
   
   All columns in ATTRIBUTES must also appear in the source SELECT:
   ```sql
   -- Wrong: region is in ATTRIBUTES but not in SELECT
   CREATE CORTEX SEARCH SERVICE my_service
     ON text_col
     ATTRIBUTES region
     AS (SELECT text_col FROM my_table);  -- Missing region!
   
   -- Correct
   CREATE CORTEX SEARCH SERVICE my_service
     ON text_col
     ATTRIBUTES region
     AS (SELECT text_col, region FROM my_table);
   ```

### Error: "Invalid filter syntax"

**Symptoms:**
```
Invalid filter object
```
or empty/unexpected results

**Causes & Solutions:**

1. **Missing @ prefix on operators**
   ```json
   // Wrong
   {"eq": {"region": "North America"}}
   
   // Correct
   {"@eq": {"region": "North America"}}
   ```

2. **@and/@or takes an array, not an object**
   ```json
   // Wrong
   {"@and": {"@eq": {"region": "NA"}, "@eq": {"category": "Billing"}}}
   
   // Correct
   {"@and": [{"@eq": {"region": "NA"}}, {"@eq": {"category": "Billing"}}]}
   ```

3. **Invalid JSON in SEARCH_PREVIEW**
   ```sql
   -- Wrong: unescaped quotes or malformed JSON
   SNOWFLAKE.CORTEX.SEARCH_PREVIEW('svc', '{"query": "test", "filter": {bad json}}')
   
   -- Correct: valid JSON string
   SNOWFLAKE.CORTEX.SEARCH_PREVIEW('svc', '{"query": "test", "filter": {"@eq": {"region": "NA"}}}')
   ```

4. **Using @contains on a non-array column**
   
   `@contains` is for array-type columns only. For string equality, use `@eq`.

### Error: "Service not found"

**Symptoms:**
```
Cortex Search Service 'MY_SERVICE' does not exist
```

**Causes & Solutions:**

1. **Wrong schema context**
   ```sql
   -- Check current schema
   SELECT CURRENT_DATABASE(), CURRENT_SCHEMA();
   
   -- Use fully qualified name
   SNOWFLAKE.CORTEX.SEARCH_PREVIEW(
       'my_db.my_schema.my_service',
       '...'
   )
   ```

2. **Case sensitivity with double-quoted names**
   
   If the service was created with double quotes (e.g., in Snowsight), the name is case-sensitive:
   ```sql
   -- If created as "My Service", you must reference it exactly
   SNOWFLAKE.CORTEX.SEARCH_PREVIEW('"My Service"', '...')
   ```

3. **Service was dropped or replaced**
   ```sql
   -- Check if service exists
   SHOW CORTEX SEARCH SERVICES;
   ```

## Service Not Refreshing

### Problem: Data is stale

**Diagnosis:**
```sql
-- Check service refresh status
SHOW CORTEX SEARCH SERVICES;
-- Look at the last_refreshed_at column
```

**Causes & Solutions:**

1. **Change tracking not enabled**
   ```sql
   -- Enable change tracking for incremental refresh
   ALTER TABLE my_table SET CHANGE_TRACKING = TRUE;
   ```

2. **TARGET_LAG too long**
   ```sql
   -- Reduce lag for fresher data
   ALTER CORTEX SEARCH SERVICE my_service SET TARGET_LAG = '10 minutes';
   ```

3. **Warehouse suspended**
   
   The warehouse specified in the service must be available for refreshes:
   ```sql
   ALTER WAREHOUSE my_wh RESUME;
   ```

4. **Source query error**
   
   If the source query fails during refresh (e.g., table was dropped), the service won't update. Check that the source tables/views still exist.

## Empty Results

### Problem: Search returns no results

**Diagnosis:**
```sql
-- Check if the service has data
SELECT COUNT(*) FROM TABLE(
    CORTEX_SEARCH_DATA_SCAN(SERVICE_NAME => 'my_service')
);
```

**Causes & Solutions:**

1. **Service index is empty**
   - Verify data exists in the source table
   - Check that the service was created successfully
   - Wait for the initial index build to complete

2. **Filter too restrictive**
   ```json
   // Try without filters first
   {"query": "test query", "columns": ["transcript_text"], "limit": 5}
   
   // Then add filters back one at a time
   ```

3. **Query doesn't match any content**
   
   Try a broader query or different terms. Remember that Cortex Search uses hybrid search, so both semantic meaning and keywords matter.

4. **Wrong columns requested**
   ```json
   // Make sure requested columns exist in the service
   {"query": "test", "columns": ["nonexistent_column"], "limit": 5}
   ```

## SEARCH_PREVIEW Specific Issues

### Problem: SEARCH_PREVIEW returns an error

1. **Not a string literal**
   
   SEARCH_PREVIEW only accepts string literals, not variables or expressions:
   ```sql
   -- Wrong: Using a variable
   SET my_query = '{"query": "test"}';
   SELECT SNOWFLAKE.CORTEX.SEARCH_PREVIEW('svc', $my_query);
   
   -- Correct: String literal
   SELECT SNOWFLAKE.CORTEX.SEARCH_PREVIEW('svc', '{"query": "test", "columns": ["col1"], "limit": 5}');
   ```

2. **Missing required fields**
   
   The JSON must include at least a `query` field:
   ```json
   // Minimum valid query
   {"query": "search text"}
   ```

### Problem: SEARCH_PREVIEW is slow

SEARCH_PREVIEW has higher latency than the Python and REST APIs. It is intended for testing and validation, not production use.

- For production: Use the Python SDK or REST API
- For testing: SEARCH_PREVIEW is fine

## Python SDK Issues

### Error: "Module not found"

```python
# Wrong: Old import path
from snowflake.cortex import search

# Correct: Use snowflake.core
from snowflake.core import Root
```

**Fix**: Install or upgrade the Snowflake Python APIs:
```bash
pip install snowflake -U
```

Requires version 0.8.0 or later.

### Error: "Session not active"

```python
# In Snowsight / Streamlit in Snowflake
from snowflake.snowpark.context import get_active_session
session = get_active_session()

# In local development
from snowflake.snowpark import Session
connection_params = {
    "account": "...",
    "user": "...",
    "password": "...",
}
session = Session.builder.configs(connection_params).create()
```

### Error: "Service not found" in Python

Make sure you're using the correct database, schema, and service name:

```python
# Check current context
print(session.get_current_database())
print(session.get_current_schema())

# Use explicit path
service = (
    root.databases["MY_DB"]
    .schemas["MY_SCHEMA"]
    .cortex_search_services["MY_SERVICE"]
)
```

## Performance Issues

### Slow Service Creation

**Solutions:**

1. **Use a larger warehouse** for the initial build
   ```sql
   ALTER WAREHOUSE my_wh SET WAREHOUSE_SIZE = 'MEDIUM';
   ```

2. **Reduce data volume** if possible
   - Filter the source query to relevant data
   - Remove unnecessary columns from the SELECT

3. **Use INITIALIZE = ON_SCHEDULE** to defer the first build
   ```sql
   CREATE CORTEX SEARCH SERVICE my_service
     ...
     INITIALIZE = ON_SCHEDULE  -- Don't build immediately
     ...
   ```

### Slow Search Queries

1. **Add filters** to narrow the search space
2. **Reduce limit** — requesting fewer results is faster
3. **Use the Python SDK or REST API** instead of SEARCH_PREVIEW for production

### High Cost

1. **Increase TARGET_LAG** to reduce refresh frequency
   ```sql
   ALTER CORTEX SEARCH SERVICE my_service SET TARGET_LAG = '1 day';
   ```

2. **Use a smaller warehouse** for refreshes
3. **Suspend unused services**
   ```sql
   ALTER CORTEX SEARCH SERVICE my_service SUSPEND;
   -- Resume when needed
   ALTER CORTEX SEARCH SERVICE my_service RESUME;
   ```

4. **Limit data scope** in the source query
