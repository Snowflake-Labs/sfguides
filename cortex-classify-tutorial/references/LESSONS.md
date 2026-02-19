# Cortex Classify Text Tutorial - Lesson Reference

This file contains all code for the tutorial, organized by lesson. Execute each step one at a time, explaining before running.

---

## Lesson 1: Setup and Data Loading

**Learning Objective**: Set up the environment and load sample customer review data.

### Step 1.1: Environment Setup

First, check if SNOWFLAKE_LEARNING environment exists:

```sql
-- Check for learning environment
SHOW ROLES LIKE 'SNOWFLAKE_LEARNING_ROLE';
```

**If SNOWFLAKE_LEARNING_ROLE exists** (preferred):
```sql
USE ROLE SNOWFLAKE_LEARNING_ROLE;
USE DATABASE SNOWFLAKE_LEARNING_DB;
USE WAREHOUSE SNOWFLAKE_LEARNING_WH;
```

**If NOT available** (fallback):
```sql
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
CREATE DATABASE IF NOT EXISTS LEARNING_DB;
USE DATABASE LEARNING_DB;
```

### Step 1.2: Create User Schema

```sql
-- Create a schema for this tutorial (named after current user)
SET schema_name = CONCAT(CURRENT_USER(), '_CORTEX_CLASSIFY');
CREATE SCHEMA IF NOT EXISTS IDENTIFIER($schema_name);
USE SCHEMA IDENTIFIER($schema_name);
```

**Explain**: We create a user-specific schema so multiple people can run this tutorial without conflicts.

### Step 1.3: Create File Format and Stage

```sql
-- Create file format for CSV files
CREATE OR REPLACE FILE FORMAT csv_ff 
  TYPE = 'csv';
```

**Explain**: This tells Snowflake how to parse the CSV files we'll load.

```sql
-- Create stage pointing to sample data
CREATE OR REPLACE STAGE s3load
  COMMENT = 'Tasty Bytes Voice of Customer Data'
  URL = 's3://sfquickstarts/tastybytes-voc/'
  FILE_FORMAT = csv_ff;
```

**Explain**: A stage is like a bookmark to cloud storage. This points to Snowflake's public sample data bucket.

### Step 1.4: Create Reviews Table

```sql
-- Create table for truck reviews
CREATE OR REPLACE TABLE truck_reviews
(
    order_id NUMBER(38,0),
    language VARCHAR(16777216),
    source VARCHAR(16777216),
    review VARCHAR(16777216),
    review_id NUMBER(18,0)
);
```

**Explain**: This table will hold customer reviews from various sources. Each review has an ID, the review text, language, and source (where the feedback came from).

### Step 1.5: Load the Data

```sql
-- Load reviews from S3
COPY INTO truck_reviews
FROM @s3load/raw_support/truck_reviews/;
```

**Explain**: This loads the customer reviews from S3 into our table.

### Step 1.6: Preview the Data

```sql
-- See what we're working with
SELECT * FROM truck_reviews LIMIT 10;
```

**Explain**: Let's look at the data. Notice the `review` column contains unstructured text - this is what we'll classify.

```sql
-- Count total reviews
SELECT COUNT(*) AS total_reviews FROM truck_reviews;
```

---

## Lesson 2: Classify a Single String (Python)

**Learning Objective**: Use Cortex classify_text in Python to classify a single review.

### Step 2.1: Import Python Packages

```python
# Import required packages
import streamlit as st
import pandas as pd

# Snowpark
from snowflake.snowpark.context import get_active_session
import snowflake.snowpark.functions as F

# Cortex Functions
import snowflake.cortex as cortex

# Get the active session
session = get_active_session()
```

**Explain**: We import Snowpark for data handling and the Cortex module for AI functions. `get_active_session()` connects to Snowflake.

### Step 2.2: Define a Sample Review

```python
# A positive review to classify
review = """Absolutely loved my order from Smoky BBQ in Boston! I went with the 
classic fried pickles, and let me tell you - they did not disappoint. The crispy, 
flavorful coating perfectly complemented the tangy pickles, and I couldn't get 
enough. The portion was generous, and the service was quick and friendly. If 
you're in the mood for some delicious BBQ, Smoky BBQ is the way to go. 
Highly recommend!"""

print(review)
```

**Explain**: This is a sample customer review. Reading it, we can tell this customer is happy and would likely recommend the truck. Let's see if the AI agrees.

### Step 2.3: Define Categories

```python
# Categories for classification
categories = ["Likely", "Unlikely", "Unsure"]
```

**Explain**: These are the possible outcomes. We're asking: "Is this customer likely to recommend the food truck?"

### Step 2.4: Classify the Review

```python
# Classify the review
result = cortex.classify_text(review, categories=categories)
print(result)
```

**Explain**: `classify_text` sends the review to a Cortex LLM, which analyzes the sentiment and returns the most likely category. Let's see what it says.

### Step 2.5: Understanding the Result

The result includes:
- **label**: The chosen category (e.g., "Likely")
- **score**: Confidence score (0-1)

```python
# Extract just the label
print(f"Classification: {result['label']}")
print(f"Confidence: {result['score']:.2%}")
```

---

## Lesson 3: Classify an Entire DataFrame Column

**Learning Objective**: Add a classification column to the entire reviews dataset using a task description.

### Step 3.1: Load Data into DataFrame

```python
# Load the reviews table into a Snowpark DataFrame
reviews_df = session.table('truck_reviews')

# Show first few rows
reviews_df.show(5)
```

**Explain**: We load the table into a Snowpark DataFrame, which lets us use Python operations while keeping data in Snowflake.

### Step 3.2: Define Task Description

```python
# Task description helps the LLM understand exactly what we're asking
task_description = """
Tell me based on the following food truck customer review, will they recommend 
the food truck to their friends and family?
"""
```

**Explain**: A task description gives the LLM context. Without it, the LLM might not know what aspect to classify. With it, we're explicitly asking about recommendations.

### Step 3.3: Add Classification Column

```python
# Define categories
categories = ["Likely", "Unlikely", "Unsure"]

# Add RECOMMEND column using classify_text
reviews_df = reviews_df.withColumn(
    'RECOMMEND', 
    cortex.classify_text(
        F.col('REVIEW'), 
        categories, 
        task_description
    )['label']
)
```

**Explain**: We use `withColumn` to add a new column called RECOMMEND. For each row, it classifies the REVIEW text and extracts just the label.

### Step 3.4: View Results

```python
# Show reviews with their classifications
reviews_df.select(["REVIEW", "RECOMMEND"]).show(10, max_width=100)
```

**Explain**: Now every review has a RECOMMEND classification. Let's see the distribution.

### Step 3.5: Analyze Distribution

```python
# Count by recommendation
reviews_df.group_by("RECOMMEND").count().show()
```

**Explain**: This shows how many reviews fall into each category. A healthy business should see mostly "Likely" recommendations.

### Step 3.6: Save Results (Optional)

```python
# Save classified results to a new table
reviews_df.write.mode("overwrite").save_as_table("classified_reviews")
print("Results saved to CLASSIFIED_REVIEWS table")
```

---

## Lesson 4: Classify in SQL

**Learning Objective**: Use SNOWFLAKE.CORTEX.CLASSIFY_TEXT directly in SQL.

### Step 4.1: Basic SQL Classification

```sql
-- Classify reviews directly in SQL
SELECT 
  REVIEW_ID,
  REVIEW,
  SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
    REVIEW,
    ['Likely', 'Unlikely', 'Unsure']
  ) AS classification_result
FROM truck_reviews
LIMIT 5;
```

**Explain**: In SQL, we call SNOWFLAKE.CORTEX.CLASSIFY_TEXT with the text column and an array of categories. The result is a JSON object with label and score.

### Step 4.2: Extract Just the Label

```sql
-- Extract the label from the result
SELECT 
  REVIEW_ID,
  REVIEW,
  SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
    REVIEW,
    ['Likely', 'Unlikely', 'Unsure']
  ):label::STRING AS recommend
FROM truck_reviews
LIMIT 10;
```

**Explain**: We use `:label::STRING` to extract just the classification label from the JSON result.

### Step 4.3: Add Task Description in SQL

```sql
-- With task description for better accuracy
SELECT 
  REVIEW_ID,
  REVIEW,
  CAST(
    SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
      REVIEW,
      ['Likely', 'Unlikely', 'Unsure'],
      {'task_description': 'Tell me based on the following food truck customer review, will they recommend the food truck to their friends and family?'}
    ):"label" AS STRING
  ) AS recommend
FROM truck_reviews
LIMIT 15;
```

**Explain**: The third parameter is an object with `task_description`. This helps the LLM understand exactly what we're asking, just like in Python.

### Step 4.4: Create Classified View

```sql
-- Create a view with classifications
CREATE OR REPLACE VIEW classified_reviews_v AS
SELECT 
  REVIEW_ID,
  ORDER_ID,
  LANGUAGE,
  SOURCE,
  REVIEW,
  CAST(
    SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
      REVIEW,
      ['Likely', 'Unlikely', 'Unsure'],
      {'task_description': 'Based on this food truck customer review, will they recommend the food truck to friends and family?'}
    ):"label" AS STRING
  ) AS recommend
FROM truck_reviews;
```

**Explain**: Creating a view means the classification runs on-demand when you query the view. Good for always-fresh results, but costs compute each time.

### Step 4.5: Analyze Results

```sql
-- Distribution of recommendations
SELECT 
  recommend,
  COUNT(*) AS count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 1) AS percentage
FROM classified_reviews_v
GROUP BY recommend
ORDER BY count DESC;
```

**Explain**: This shows the breakdown of recommendations. What percentage of customers would recommend the food trucks?

---

## Final Verification

```sql
-- Verify everything works
SELECT 
  'Total Reviews' AS metric, 
  COUNT(*)::STRING AS value 
FROM truck_reviews
UNION ALL
SELECT 
  'Classified Reviews', 
  COUNT(*)::STRING 
FROM classified_reviews;
```

```sql
-- Show sample classified results
SELECT review_id, 
       LEFT(review, 80) || '...' AS review_preview, 
       recommend
FROM classified_reviews
LIMIT 10;
```

---

## Summary

| What You Learned | How |
|------------------|-----|
| Classify single string | `cortex.classify_text(text, categories)` |
| Classify DataFrame column | `withColumn('COL', cortex.classify_text(...))` |
| Add task description | Pass as third parameter |
| Classify in SQL | `SNOWFLAKE.CORTEX.CLASSIFY_TEXT(col, array, options)` |
| Extract label in SQL | `:label::STRING` or `:"label"` |
