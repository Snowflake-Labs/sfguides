# Notebook Content Reference

This document contains the full content of the Cortex Classify Text tutorial notebook for reference.

## Overview

The notebook teaches users to classify unstructured customer reviews using Snowflake Cortex. It covers:

1. Setting up the environment and loading sample data
2. Classifying a single text string with Python
3. Classifying an entire DataFrame column
4. Classifying directly in SQL

## Cell-by-Cell Breakdown

### Cell 1: Overview (Markdown)

Introduces the tutorial:
- Explains the business value of understanding customer feedback
- Introduces Tasty Bytes (fictional food truck company)
- Sets up the learning objectives

### Cell 2: Import Sample Data (SQL)

```sql
USE ROLE SNOWFLAKE_LEARNING_ROLE;
USE DATABASE SNOWFLAKE_LEARNING_DB;

SET schema_name = CONCAT(current_user(), '_CLASSIFY_UNSTRUCTURED_CUSTOMER_REVIEWS');
CREATE SCHEMA IF NOT EXISTS IDENTIFIER($schema_name);
USE SCHEMA IDENTIFIER($schema_name);

-- File format and stage
CREATE OR REPLACE FILE FORMAT csv_ff TYPE = 'csv';

CREATE OR REPLACE STAGE s3load
  COMMENT = 'Quickstarts S3 Stage Connection'
  URL = 's3://sfquickstarts/tastybytes-voc/'
  FILE_FORMAT = csv_ff;

-- Table creation
CREATE OR REPLACE TABLE truck_reviews (
    order_id NUMBER(38,0),
    language VARCHAR(16777216),
    source VARCHAR(16777216),
    review VARCHAR(16777216),
    review_id NUMBER(18,0)
);

-- Load data
COPY INTO truck_reviews FROM @s3load/raw_support/truck_reviews/;

SELECT 'Setup is complete' AS note;
```

### Cell 3: Import Python Packages

```python
import streamlit as st
import pandas as pd
from snowflake.snowpark.context import get_active_session
import snowflake.snowpark.functions as F
import snowflake.cortex as cortex

session = get_active_session()
```

### Cell 4: Preview Table (SQL)

```sql
SELECT * FROM TRUCK_REVIEWS LIMIT 15;
```

### Cell 5: Classify Single String (Python)

```python
review = """Absolutely loved my order from Smoky BBQ in Boston! I went with the 
classic fried pickles, and let me tell you - they did not disappoint. The crispy, 
flavorful coating perfectly complemented the tangy pickles, and I couldn't get 
enough. The portion was generous, and the service was quick and friendly. If 
you're in the mood for some delicious BBQ, Smoky BBQ is the way to go. 
Highly recommend!"""

categories = ["Likely", "Unlikely", "Unsure"]

cortex.classify_text(review, categories=categories)
```

**Expected Output:**
```python
{'label': 'Likely', 'score': 0.95}
```

### Cell 6: Classify DataFrame Column (Python)

```python
task_description = """
Tell me based on the following food truck customer review, will they recommend 
the food truck to their friends and family?
"""

reviews_df = session.table('truck_reviews')

reviews_df = reviews_df.withColumn(
    'RECOMMEND', 
    cortex.classify_text(F.col('REVIEW'), categories, task_description)['label']
)

reviews_df.select(["REVIEW", "RECOMMEND"]).show(15, max_width=125)
```

### Cell 7: Classify in SQL

```sql
SELECT 
  REVIEW_ID,
  REVIEW,
  CAST(
    SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
      REVIEW,
      ['Likely','Unlikely','Unsure'],
      {'task_description': 'Tell me based on the following food truck customer review, will they recommend the food truck to their friends and family?'}
    ):"label" AS STRING
  ) AS RECOMMEND
FROM TRUCK_REVIEWS
LIMIT 15;
```

### Cell 8: Conclusion (Markdown)

Summarizes what was learned and provides links to documentation:
- Cortex Classify Text docs
- Cortex LLM Functions docs

## Key Concepts Demonstrated

| Concept | Where Demonstrated |
|---------|-------------------|
| Zero-shot classification | All classification cells |
| Python cortex.classify_text | Cells 5, 6 |
| SQL CLASSIFY_TEXT | Cell 7 |
| Task descriptions | Cells 6, 7 |
| DataFrame operations | Cell 6 |
| JSON extraction in SQL | Cell 7 (`:label::STRING`) |

## Data Used

**Source:** `s3://sfquickstarts/tastybytes-voc/raw_support/truck_reviews/`

**Schema:**
- `order_id`: Order identifier
- `language`: Language of review
- `source`: Where feedback came from
- `review`: The actual review text
- `review_id`: Unique review identifier

## Environment Requirements

- SNOWFLAKE_LEARNING_ROLE (or equivalent)
- SNOWFLAKE_LEARNING_DB (or user-created database)
- Warehouse with Cortex access
- Cortex AI enabled in account

---

## Getting the Latest Documentation

### Official Snowflake Documentation

| Topic | URL |
|-------|-----|
| CLASSIFY_TEXT Function | https://docs.snowflake.com/en/sql-reference/functions/classify_text-snowflake-cortex |
| Cortex LLM Functions | https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions |
| Snowflake Notebooks | https://docs.snowflake.com/en/user-guide/ui-snowsight/notebooks |
| Creating Notebooks | https://docs.snowflake.com/en/user-guide/ui-snowsight/notebooks-create |

### How to Fetch Latest Docs

Ask Cortex Code to fetch current documentation:

```
"Fetch the latest CLASSIFY_TEXT docs"
"Get the current Snowflake Notebooks documentation"
```

### SNOWFLAKE_LEARNING Environment

| Resource | URL |
|----------|-----|
| Learning Environment (BCR-1992) | https://docs.snowflake.com/en/release-notes/bcr-bundles/un-bundled/bcr-1992 |
| Snowsight Templates | https://docs.snowflake.com/en/user-guide/ui-snowsight/snowsight-templates |

### Release Notes

| Resource | URL |
|----------|-----|
| Snowflake Release Notes | https://docs.snowflake.com/en/release-notes |
| Cortex AI Features | https://docs.snowflake.com/en/release-notes/new-features#cortex-ai |
