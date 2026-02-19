# Troubleshooting CLASSIFY_TEXT

## Common Errors and Solutions

### Error: "Function CLASSIFY_TEXT does not exist"

**Symptoms:**
```
SQL compilation error: Unknown function SNOWFLAKE.CORTEX.CLASSIFY_TEXT
```

**Causes & Solutions:**

1. **Cortex not enabled in region**
   ```sql
   -- Check if Cortex functions are available
   SELECT SNOWFLAKE.CORTEX.COMPLETE('snowflake-arctic', 'Hello') AS test;
   ```
   If this fails, Cortex may not be available in your region.

2. **Wrong function name**
   ```sql
   -- Wrong
   SELECT CLASSIFY_TEXT(text, categories);
   
   -- Correct (full path required)
   SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, categories);
   ```

3. **Insufficient privileges**
   - Need USAGE on SNOWFLAKE database
   - Need appropriate Cortex permissions

### Error: "Invalid categories array"

**Symptoms:**
```
Categories must be a non-empty array of strings
```

**Causes & Solutions:**

1. **Empty array**
   ```sql
   -- Wrong
   SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, []);
   
   -- Correct
   SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, ['A', 'B']);
   ```

2. **Single category**
   ```sql
   -- Wrong (need at least 2)
   SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, ['Positive']);
   
   -- Correct
   SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, ['Positive', 'Negative']);
   ```

3. **Wrong syntax in SQL**
   ```sql
   -- Wrong (Python list syntax)
   SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, ["A", "B"]);
   
   -- Correct (single quotes in SQL)
   SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, ['A', 'B']);
   ```

### Error: "Text too long"

**Symptoms:**
```
Input text exceeds maximum token limit
```

**Solutions:**

```python
# Truncate long text
max_length = 4000  # characters
df = df.withColumn('truncated_text', 
    F.substring(F.col('text'), 1, max_length))
```

```sql
-- Truncate in SQL
SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
    LEFT(long_text, 4000), categories);
```

### Error: "NULL input"

**Symptoms:**
- Returns NULL for some rows
- Inconsistent results

**Causes & Solutions:**

```sql
-- Filter out NULLs before classifying
SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, categories)
FROM my_table
WHERE text IS NOT NULL AND text != '';
```

```python
# Python: Filter NULLs
df = df.filter(F.col('text').isNotNull())
df = df.filter(F.length(F.col('text')) > 0)
```

### Error: "Rate limit exceeded"

**Symptoms:**
```
Rate limit exceeded for Cortex functions
```

**Solutions:**

1. **Add delay between calls**
   ```python
   import time
   for batch in batches:
       result = process_batch(batch)
       time.sleep(1)  # Add delay
   ```

2. **Reduce batch size**
   ```python
   # Process in smaller chunks
   df.limit(1000).withColumn('category', cortex.classify_text(...))
   ```

3. **Contact Snowflake for rate limit increase**

### Error: "Warehouse timeout"

**Symptoms:**
```
Query execution timeout
```

**Solutions:**

1. **Use larger warehouse**
   ```sql
   USE WAREHOUSE LARGE_WH;
   ```

2. **Process in batches**
   ```sql
   -- Process subset
   SELECT ... FROM my_table LIMIT 10000;
   ```

3. **Increase statement timeout**
   ```sql
   ALTER SESSION SET STATEMENT_TIMEOUT_IN_SECONDS = 3600;
   ```

## Poor Classification Quality

### Problem: Random-seeming results

**Diagnosis:**
```python
# Check confidence scores
result = cortex.classify_text(text, categories)
print(f"Label: {result['label']}, Score: {result['score']}")
# Low scores (<0.5) indicate uncertainty
```

**Solutions:**

1. **Add task description**
   ```python
   # Without: LLM guesses what you want
   cortex.classify_text(text, categories)
   
   # With: LLM knows exactly what to classify
   cortex.classify_text(text, categories, 
       task_description="Classify the customer's overall satisfaction")
   ```

2. **Improve categories**
   ```python
   # Bad: Overlapping categories
   categories = ["Good", "Great", "Excellent"]
   
   # Good: Distinct categories
   categories = ["Positive", "Neutral", "Negative"]
   ```

3. **Add catch-all category**
   ```python
   # Add "Unsure" for ambiguous text
   categories = ["Positive", "Negative", "Unsure"]
   ```

### Problem: Wrong aspect being classified

**Example:** Review says "Terrible shipping, amazing product" → classified as "Negative"

**Solution: Specify aspect in task description**
```python
task = """
Classify the customer's satisfaction with the PRODUCT ITSELF.
Ignore comments about shipping, packaging, or delivery.
"""
```

### Problem: Neutral text classified as positive/negative

**Solution: Add Neutral category**
```python
# Before
categories = ["Positive", "Negative"]  # Forces a choice

# After  
categories = ["Positive", "Negative", "Neutral"]  # Allows middle ground
```

### Problem: Inconsistent results for similar texts

**Diagnosis:**
```python
# Test same category multiple times
texts = [
    "Pretty good product",
    "Product is pretty good",
    "I think the product is good"
]
for t in texts:
    result = cortex.classify_text(t, categories)
    print(f"{result['label']} ({result['score']:.2f}): {t}")
```

**Solutions:**
1. Use clearer task description
2. Use more distinct categories
3. Pre-process text for consistency

## Python-Specific Issues

### Import Error

```python
# Wrong
from cortex import classify_text

# Correct
import snowflake.cortex as cortex
result = cortex.classify_text(...)
```

### Session Error

```python
# Error: No active session
from snowflake.snowpark.context import get_active_session
session = get_active_session()  # Must be in Snowflake environment
```

**In local development, create session manually:**
```python
from snowflake.snowpark import Session
connection_params = {
    "account": "...",
    "user": "...",
    "password": "...",
    # etc.
}
session = Session.builder.configs(connection_params).create()
```

### Column vs String Confusion

```python
# Wrong: Passing column name as string
df.withColumn('cat', cortex.classify_text('text_column', categories))

# Correct: Using F.col() for column reference
df.withColumn('cat', cortex.classify_text(F.col('text_column'), categories))
```

## SQL-Specific Issues

### JSON Extraction

```sql
-- Wrong: Missing type cast
SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, categories):label AS sentiment
-- Returns VARIANT type

-- Correct: Cast to STRING
SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, categories):label::STRING AS sentiment
```

### Options Object Syntax

```sql
-- Wrong: Missing quotes on key
SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, cats, {task_description: '...'});

-- Correct: Quoted key
SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, cats, {'task_description': '...'});
```

## Performance Issues

### Slow Query on Large Table

**Solutions:**

1. **Add LIMIT for testing**
   ```sql
   SELECT ... LIMIT 100;  -- Test with small sample first
   ```

2. **Use larger warehouse**
   ```sql
   USE WAREHOUSE LARGE_WH;
   ```

3. **Process incrementally**
   ```sql
   -- Classify only new rows
   INSERT INTO classified_data
   SELECT id, text, SNOWFLAKE.CORTEX.CLASSIFY_TEXT(...)
   FROM raw_data
   WHERE id > (SELECT MAX(id) FROM classified_data);
   ```

### High Cost

**Solutions:**

1. **Cache results**: Store classifications, don't re-classify same text
2. **Sample first**: Test on sample before full table
3. **Shorter task descriptions**: Fewer tokens = lower cost
4. **Batch similar requests**: Group by classification type
