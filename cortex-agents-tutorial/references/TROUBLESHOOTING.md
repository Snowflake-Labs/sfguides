# Cortex Agents Troubleshooting Guide

Common issues and solutions when working with Cortex Agents.

**Updated: March 2026** - Based on tested, working implementations.

---

## CRITICAL: Most Common Errors

### "syntax error...unexpected 'AS'" (Semantic View)

**Cause**: Using wrong semantic view syntax (embedded YAML instead of SQL clauses).

**WRONG** (will fail):
```sql
CREATE OR REPLACE SEMANTIC VIEW my_view
  AS SELECT * FROM my_table
  WITH SEMANTIC MODEL $$
    name: my_model
    tables:
      - name: my_table
        ...
  $$;
```

**CORRECT**:
```sql
CREATE OR REPLACE SEMANTIC VIEW my_view
  TABLES (
    my_table AS DATABASE.SCHEMA.MY_TABLE
      PRIMARY KEY (id)
  )
  DIMENSIONS (
    my_table.column1 AS column1 COMMENT = 'Description'
  )
  METRICS (
    my_table.total AS SUM(amount) COMMENT = 'Total amount'
  );
```

---

### "The Analyst tool is missing an execution environment"

**Cause**: Agent's Cortex Analyst tool doesn't have a warehouse specified.

**WRONG**:
```yaml
tool_resources:
  SalesAnalyst:
    semantic_view: "DB.SCHEMA.VIEW"
    warehouse: "COMPUTE_WH"  # Wrong location!
```

**CORRECT**:
```yaml
tool_resources:
  SalesAnalyst:
    semantic_view: "DB.SCHEMA.VIEW"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
```

---

### "Request is malformed" (DATA_AGENT_RUN)

**Cause**: Using wrong format for DATA_AGENT_RUN function.

**WRONG**:
```sql
SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'DB.SCHEMA.MY_AGENT',
    'What are total sales?'  -- Wrong! Not a simple string
);
```

**CORRECT**:
```sql
SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'DB.SCHEMA.MY_AGENT',
    $${
      "messages": [
        {
          "role": "user",
          "content": [
            { "type": "text", "text": "What are total sales?" }
          ]
        }
      ]
    }$$
);
```

---

### "syntax error...unexpected 'AGENT'"

**Cause**: Using `CREATE CORTEX AGENT` instead of `CREATE AGENT`.

**WRONG**:
```sql
CREATE OR REPLACE CORTEX AGENT my_agent ...
```

**CORRECT**:
```sql
CREATE OR REPLACE AGENT my_agent ...
```

---

### Custom tools not visible in Snowsight UI

**Cause**: Created UDFs (`CREATE FUNCTION`) instead of stored procedures (`CREATE PROCEDURE`). The Snowsight custom tools picker only lists procedures.

**WRONG**:
```sql
CREATE OR REPLACE FUNCTION check_inventory(product_name VARCHAR)
RETURNS VARCHAR
LANGUAGE SQL
AS $$ ... $$;
```

**CORRECT**:
```sql
CREATE OR REPLACE PROCEDURE check_inventory_proc(p_product_name VARCHAR)
RETURNS VARCHAR
LANGUAGE SQL
EXECUTE AS CALLER
AS
$$
DECLARE
    result VARCHAR;
BEGIN
    SELECT ... INTO :result FROM ... WHERE UPPER(col) = UPPER(:p_product_name);
    RETURN result;
END;
$$;
```

**Key differences**:
- Use `CREATE PROCEDURE`, not `CREATE FUNCTION`
- Use `EXECUTE AS CALLER` for proper privilege flow
- Use `DECLARE`/`BEGIN`/`END` block with `$$` delimiters
- Prefix parameters with `p_` to avoid column name collisions
- Reference parameters with colon prefix (`:p_product_name`) inside SQL statements

---

### "invalid identifier 'FUNCTION_NAME.PARAMETER_NAME'" in SQL UDFs

**Cause**: In SQL UDFs, disambiguating parameters with `function_name.param` (e.g., `check_inventory.product_name`) causes compilation errors.

**Solution**: Use stored procedures with `p_` prefixed parameter names instead. See "Custom tools not visible in Snowsight UI" above.

---

## Agent Creation Errors

### "Insufficient privileges to create agent"

**Cause**: Role lacks CREATE AGENT privilege.

**Solution**:
```sql
GRANT CREATE AGENT ON SCHEMA mydb.myschema TO ROLE my_role;

-- Or use ACCOUNTADMIN
USE ROLE ACCOUNTADMIN;
```

### "Invalid specification format"

**Cause**: YAML syntax error in agent spec.

**Solution**:
1. Check indentation (YAML is indent-sensitive)
2. Ensure property names don't have quotes unless needed
3. Use online YAML validator

Common mistakes:
```yaml
# Wrong - inconsistent indentation
tools:
  - tool_spec:
      type: "cortex_analyst_text_to_sql"
    name: "SalesAnalyst"  # Wrong indent!

# Correct
tools:
  - tool_spec:
      type: "cortex_analyst_text_to_sql"
      name: "SalesAnalyst"
```

### "Model not available in region"

**Cause**: Requested model isn't available in your Snowflake region.

**Solution**:
1. Enable cross-region inference in account settings
2. Or omit the model to use auto-selection

---

## Tool Configuration Errors

### "Semantic view not found"

**Cause**: Wrong path or view doesn't exist.

**Solution**:
```sql
-- Verify semantic view exists
SHOW SEMANTIC VIEWS IN SCHEMA mydb.myschema;

-- Use UPPERCASE for identifier (Snowflake default)
-- In agent spec:
semantic_view: "MYDB.MYSCHEMA.MY_SEMANTIC_VIEW"
```

### "Search service not found"

**Cause**: Wrong path or using wrong key name.

**Solution**:
```sql
-- Verify search service exists
SHOW CORTEX SEARCH SERVICES IN SCHEMA mydb.myschema;
```

**In agent spec, use `name` not `search_service`:**
```yaml
tool_resources:
  ProductDocs:
    name: "MYDB.MYSCHEMA.PRODUCT_SEARCH_SERVICE"  # Use 'name'!
    max_results: 5
```

### "Missing tool_resources for tool"

**Cause**: Tool defined in `tools` but not configured in `tool_resources`.

**Solution**: Ensure every tool has a matching entry with EXACT same name:
```yaml
tools:
  - tool_spec:
      name: "SalesAnalyst"  # This name...
      ...

tool_resources:
  SalesAnalyst:  # ...must match this key!
    ...
```

---

## Runtime Errors

### "Agent not responding" / Timeout

**Causes**:
- Warehouse not running
- Query timeout too short
- Complex query taking too long

**Solutions**:
```sql
-- Ensure warehouse is running
ALTER WAREHOUSE COMPUTE_WH RESUME;

-- In agent spec, increase timeout:
execution_environment:
  type: "warehouse"
  warehouse: "COMPUTE_WH"
  query_timeout: 120  # seconds
```

### "Permission denied" when running agent

**Cause**: User lacks USAGE on agent or underlying resources.

**Solution**:
```sql
-- Grant agent usage
GRANT USAGE ON AGENT mydb.myschema.agent TO ROLE user_role;

-- User must also have default role and warehouse set
ALTER USER myuser SET DEFAULT_ROLE = my_role;
ALTER USER myuser SET DEFAULT_WAREHOUSE = COMPUTE_WH;
```

### "Tool execution failed"

**Causes**:
- Underlying semantic view has errors
- Search service is still initializing
- Column referenced doesn't exist

**Solutions**:
1. Test the semantic view directly:
   ```sql
   SELECT * FROM SEMANTIC_VIEW(
       MY_SEMANTIC_VIEW
       METRICS total_sales
       DIMENSIONS region
   );
   ```
2. Check search service status:
   ```sql
   DESC CORTEX SEARCH SERVICE my_search;
   ```
3. Verify table columns:
   ```sql
   DESCRIBE TABLE my_table;
   ```

---

## Semantic View Errors

### "invalid identifier 'COLUMN_NAME'"

**Cause**: Column referenced in semantic view doesn't exist in base table.

**Solution**:
```sql
-- Check actual column names
DESCRIBE TABLE my_table;

-- Use exact column names (case-sensitive when quoted)
```

### Query returns unexpected results

**Cause**: Semantic view metrics/dimensions don't match expectations.

**Solution**: Test semantic view directly:
```sql
SELECT * FROM SEMANTIC_VIEW(
    DATABASE.SCHEMA.MY_VIEW
    METRICS metric1, metric2
    DIMENSIONS dim1, dim2
);
```

---

## Search Service Errors

### "Search service is initializing"

**Cause**: Service was just created and is still indexing.

**Solution**: Wait for initialization:
```sql
DESC CORTEX SEARCH SERVICE my_search;
-- Wait until ready
```

### "No results returned"

**Causes**:
- Query doesn't match indexed content
- Wrong column specified for `ON` clause

**Solutions**:
1. Test with broader query
2. Verify source data exists:
   ```sql
   SELECT * FROM source_table LIMIT 10;
   ```

---

## Debugging Tips

### Test Components Independently

Before debugging the agent:
1. Test semantic view: `SELECT * FROM SEMANTIC_VIEW(...)`
2. Test search service: Check via `DESC`
3. Test procedure: `CALL my_proc(...)`

### Check Agent Response Details

The DATA_AGENT_RUN response contains a JSON `content` array with items of type:
- `thinking` - Agent's reasoning (`item["thinking"]["text"]`)
- `tool_use` - Which tools were called (`item["tool_use"]["name"]`)
- `tool_result` - Results from each tool (SQL, result_set, etc.)
- `text` - Final answer

Parse the response with Python for full visibility:
```python
import json
from snowflake.snowpark.context import get_active_session

session = get_active_session()
result = session.sql("""
  SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'DB.SCHEMA.MY_AGENT',
    $${"messages": [{"role": "user", "content": [{"type": "text", "text": "Your question"}]}]}$$
  ) AS resp
""").collect()

resp = json.loads(result[0]["RESP"])
for item in resp.get("content", []):
    print(f"--- {item.get('type')} ---")
    if item.get("type") == "thinking":
        print(item["thinking"]["text"])
    elif item.get("type") == "tool_result":
        content_json = item["tool_result"].get("content", [{}])[0].get("json", {})
        if content_json.get("sql"):
            print(f"SQL: {content_json['sql']}")
        if content_json.get("result_set", {}).get("data"):
            print(f"Data: {content_json['result_set']['data']}")
    elif item.get("type") == "text":
        print(item["text"])
```

**Note**: Raw SQL output (without Python parsing) will truncate or hide the thinking, generated SQL, and result set details. Always use Python `json.loads()` for debugging.

### Simplify and Iterate

1. Start with minimal spec (one tool)
2. Verify it works
3. Add tools one at a time
4. Test after each addition

---

## Quick Reference: Correct Patterns

| Component | Correct Pattern |
|-----------|-----------------|
| Semantic View | `TABLES (...) DIMENSIONS (...) METRICS (...)` |
| Agent Creation | `CREATE AGENT ... FROM SPECIFICATION $$ yaml $$` |
| Analyst tool_resources | `execution_environment: {type: warehouse, warehouse: WH}` |
| Search tool_resources | `name: "DB.SCHEMA.SERVICE"` |
| DATA_AGENT_RUN | JSON with `messages` array |

---

## Getting Help

- **Documentation**: https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents
- **CREATE AGENT Syntax**: https://docs.snowflake.com/en/sql-reference/sql/create-agent
- **Semantic Views**: https://docs.snowflake.com/en/sql-reference/sql/create-semantic-view
- **Community**: https://community.snowflake.com
