# Python vs SQL: When to Use Each

Both Python (Snowpark) and SQL can use CLASSIFY_TEXT. Here's how to choose.

## Quick Decision Guide

| Scenario | Use |
|----------|-----|
| Exploring/prototyping | Python |
| One-off analysis | Either |
| Production pipeline | SQL |
| Complex pre-processing | Python |
| Integration with ML workflow | Python |
| Views/scheduled queries | SQL |
| Non-Python team | SQL |

## Python Approach

### Syntax

```python
import snowflake.cortex as cortex
from snowflake.snowpark.context import get_active_session
import snowflake.snowpark.functions as F

session = get_active_session()

# Single string
result = cortex.classify_text(text, categories, task_description)

# DataFrame column
df = df.withColumn('category', 
    cortex.classify_text(F.col('text'), categories, task_description)['label'])
```

### Strengths

1. **Interactive exploration**: See results immediately in notebooks
2. **Complex logic**: Pre-process text, conditional classification
3. **ML integration**: Works with Snowpark ML, pandas, sklearn
4. **Rich output**: Easy access to both label and score
5. **Iteration**: Quick to modify and re-run

### Example: Multi-Step Processing

```python
# Complex preprocessing in Python
df = session.table('reviews')

# Clean text
df = df.withColumn('cleaned_text', 
    F.regexp_replace(F.col('text'), r'<[^>]+>', ''))  # Remove HTML

# Only classify English reviews
df = df.filter(F.col('language') == 'en')

# Classify with confidence threshold
df = df.withColumn('classification',
    cortex.classify_text(F.col('cleaned_text'), categories, task))

df = df.withColumn('label', F.col('classification')['label'])
df = df.withColumn('confidence', F.col('classification')['score'])

# Route based on confidence
high_confidence = df.filter(F.col('confidence') > 0.8)
needs_review = df.filter(F.col('confidence') <= 0.8)
```

## SQL Approach

### Syntax

```sql
-- Basic
SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text_column, 
    ['Category1', 'Category2', 'Category3']) AS result
FROM my_table;

-- With task description
SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text_column,
    ['Category1', 'Category2'],
    {'task_description': 'Your task description here'}) AS result
FROM my_table;

-- Extract label
SELECT 
    text_column,
    SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text_column, 
        ['Positive', 'Negative', 'Neutral']):label::STRING AS sentiment
FROM my_table;
```

### Strengths

1. **Production ready**: Easy to schedule, monitor, and maintain
2. **Views**: Create classified views that auto-update
3. **No Python required**: Works for SQL-only teams
4. **Integration**: Works in dbt, BI tools, dashboards
5. **Simplicity**: One-line classification

### Example: Classified View

```sql
-- Create a view that classifies on-demand
CREATE OR REPLACE VIEW classified_feedback AS
SELECT 
    feedback_id,
    customer_id,
    feedback_text,
    created_at,
    SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
        feedback_text,
        ['Praise', 'Complaint', 'Question', 'Suggestion'],
        {'task_description': 'Classify customer feedback by type'}
    ):label::STRING AS feedback_type,
    SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
        feedback_text,
        ['Urgent', 'Normal', 'Low'],
        {'task_description': 'Determine urgency of this feedback'}
    ):label::STRING AS priority
FROM raw_feedback;
```

## Side-by-Side Comparison

### Single Text Classification

**Python:**
```python
result = cortex.classify_text(
    "Great product, highly recommend!", 
    ["Positive", "Negative", "Neutral"]
)
print(result['label'])  # "Positive"
```

**SQL:**
```sql
SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
    'Great product, highly recommend!',
    ['Positive', 'Negative', 'Neutral']
):label::STRING;
-- Returns: "Positive"
```

### Column Classification

**Python:**
```python
df = session.table('reviews')
df = df.withColumn('sentiment',
    cortex.classify_text(F.col('review_text'), 
        ['Positive', 'Negative', 'Neutral'])['label'])
df.show()
```

**SQL:**
```sql
SELECT 
    review_text,
    SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
        review_text,
        ['Positive', 'Negative', 'Neutral']
    ):label::STRING AS sentiment
FROM reviews;
```

### With Task Description

**Python:**
```python
task = "Based on this review, would the customer recommend the product?"
df = df.withColumn('would_recommend',
    cortex.classify_text(
        F.col('review_text'),
        ['Likely', 'Unlikely', 'Unsure'],
        task
    )['label'])
```

**SQL:**
```sql
SELECT 
    review_text,
    SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
        review_text,
        ['Likely', 'Unlikely', 'Unsure'],
        {'task_description': 'Based on this review, would the customer recommend the product?'}
    ):label::STRING AS would_recommend
FROM reviews;
```

## Hybrid Approach

Often the best approach combines both:

### Development (Python)

```python
# Explore and iterate in notebook
categories = ["Positive", "Negative", "Neutral"]
task = "Classify sentiment"

sample = session.table('reviews').limit(100)
sample = sample.withColumn('sentiment',
    cortex.classify_text(F.col('text'), categories, task)['label'])
sample.show()

# Analyze results, adjust categories/task
```

### Production (SQL)

```sql
-- Deploy as view or scheduled query
CREATE OR REPLACE VIEW production.classified_reviews AS
SELECT 
    review_id,
    text,
    SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
        text, 
        ['Positive', 'Negative', 'Neutral'],
        {'task_description': 'Classify sentiment'}
    ):label::STRING AS sentiment
FROM production.reviews;
```

## Performance Considerations

### Python (Snowpark)

- Parallelizes automatically within Snowflake
- Overhead for small datasets (session setup)
- Best for: Moderate to large datasets with complex logic

### SQL

- Native execution, minimal overhead
- Best for: Simple transformations, views, scheduled jobs

### Latency Comparison

| Operation | Python | SQL |
|-----------|--------|-----|
| Single text | ~300ms + session | ~200ms |
| 100 rows | ~500ms | ~500ms |
| 10,000 rows | ~5s | ~5s |
| 100,000 rows | ~50s | ~50s |

For large tables, both approaches parallelize similarly.

## Common Patterns

### Pattern: Store Classification Results

**Python:**
```python
df = session.table('raw_data')
df = df.withColumn('category', 
    cortex.classify_text(F.col('text'), categories)['label'])
df.write.mode('overwrite').save_as_table('classified_data')
```

**SQL:**
```sql
CREATE OR REPLACE TABLE classified_data AS
SELECT *,
    SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, 
        ['A', 'B', 'C']):label::STRING AS category
FROM raw_data;
```

### Pattern: Incremental Classification

**SQL (with Dynamic Table):**
```sql
CREATE OR REPLACE DYNAMIC TABLE classified_feedback
TARGET_LAG = '1 hour'
WAREHOUSE = COMPUTE_WH
AS
SELECT 
    id,
    feedback,
    SNOWFLAKE.CORTEX.CLASSIFY_TEXT(feedback, 
        ['Urgent', 'Normal', 'Low']):label::STRING AS priority
FROM new_feedback;
```

### Pattern: Classification in WHERE Clause

**SQL:**
```sql
-- Find all urgent feedback (expensive - classifies every row)
SELECT * FROM feedback
WHERE SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
    text, ['Urgent', 'Normal']):label::STRING = 'Urgent';
```

**Better: Classify once, filter after:**
```sql
WITH classified AS (
    SELECT *,
        SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, 
            ['Urgent', 'Normal']):label::STRING AS priority
    FROM feedback
)
SELECT * FROM classified WHERE priority = 'Urgent';
```
