# Cortex Search Tutorial - Lesson Reference

This file contains all code for the tutorial, organized by lesson. Execute each step one at a time, explaining before running.

---

## Lesson 1: Setup and Data Loading

**Learning Objective**: Set up the environment and load sample customer support transcript data.

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
SET schema_name = CONCAT(CURRENT_USER(), '_CORTEX_SEARCH');
CREATE SCHEMA IF NOT EXISTS IDENTIFIER($schema_name);
USE SCHEMA IDENTIFIER($schema_name);
```

**Explain**: We create a user-specific schema so multiple people can run this tutorial without conflicts.

### Step 1.3: Create Support Transcripts Table

```sql
-- Create table for customer support transcripts
CREATE OR REPLACE TABLE support_transcripts (
    transcript_id NUMBER(38,0),
    transcript_text VARCHAR,
    region VARCHAR,
    product_category VARCHAR,
    agent_id VARCHAR,
    created_date DATE
);
```

**Explain**: This table will hold customer support conversations. Each transcript has an ID, the conversation text, the customer's region, what product category it relates to, which agent handled it, and when it happened.

### Step 1.4: Load Sample Data

```sql
-- Insert sample support transcripts
INSERT INTO support_transcripts VALUES
    (1001, 'Customer called about internet connectivity dropping every few hours. Walked them through resetting the router and checking firmware. Issue resolved after firmware update.', 'North America', 'Internet', 'AG2001', '2024-11-01'),
    (1002, 'Customer was overcharged $15.99 on their October bill for a service they cancelled in September. Issued a refund and confirmed the cancellation was processed.', 'North America', 'Billing', 'AG2002', '2024-11-02'),
    (1003, 'Customer unable to reset password. The reset email link expired before they could use it. Sent a new reset link and extended the expiration window to 24 hours.', 'Europe', 'Account', 'AG2003', '2024-11-03'),
    (1004, 'Customer received a faulty router that would not power on. Arranged a replacement shipment with express delivery at no additional cost.', 'North America', 'Hardware', 'AG2004', '2024-11-04'),
    (1005, 'Customer asked how to upgrade their plan from Basic to Premium. Explained the pricing difference and features. Customer decided to upgrade effective next billing cycle.', 'Europe', 'Billing', 'AG2001', '2024-11-05'),
    (1006, 'Customer reported extremely slow download speeds during peak hours. Ran remote diagnostics and found network congestion in their area. Scheduled a technician visit for signal boost.', 'Asia', 'Internet', 'AG2005', '2024-11-06'),
    (1007, 'Customer wants to cancel their subscription and get a prorated refund for the remaining month. Processed cancellation and issued refund of $8.50 to their credit card.', 'North America', 'Billing', 'AG2002', '2024-11-07'),
    (1008, 'Customer could not connect their new smart TV to the Wi-Fi network. Guided them through the WPA3 settings and assigned a static IP. Device connected successfully.', 'Europe', 'Internet', 'AG2003', '2024-11-08'),
    (1009, 'Customer reported being locked out of their account after multiple failed login attempts. Verified identity and unlocked the account. Recommended enabling two-factor authentication.', 'Asia', 'Account', 'AG2004', '2024-11-09'),
    (1010, 'Customer asked about data usage limits on their current plan. Explained the 500GB monthly cap and the overage charges. Suggested the unlimited plan for heavy users.', 'North America', 'Billing', 'AG2005', '2024-11-10'),
    (1011, 'Customer complained about frequent Wi-Fi disconnections in their home office. Recommended a mesh network extender and offered a discounted equipment upgrade.', 'Europe', 'Hardware', 'AG2001', '2024-11-11'),
    (1012, 'Customer requested a detailed breakdown of their last three invoices. Emailed itemized statements and explained each line item over the phone.', 'Asia', 'Billing', 'AG2002', '2024-11-12'),
    (1013, 'Customer reported a phishing email that appeared to come from our company. Confirmed it was fraudulent, advised them not to click any links, and escalated to our security team.', 'North America', 'Account', 'AG2003', '2024-11-13'),
    (1014, 'Customer needs help setting up parental controls on their router. Walked them through the admin panel and configured content filtering for specific devices.', 'Europe', 'Internet', 'AG2004', '2024-11-14'),
    (1015, 'Customer experienced a complete service outage for over 6 hours. Confirmed a regional outage due to infrastructure maintenance. Applied a service credit to their account.', 'Asia', 'Internet', 'AG2005', '2024-11-15'),
    (1016, 'Customer asked how to return a modem they purchased but no longer need. Provided the return shipping label and explained the 30-day return policy.', 'North America', 'Hardware', 'AG2001', '2024-11-16'),
    (1017, 'Customer wants to transfer their account to a new address. Updated the service address and scheduled installation at the new location for next week.', 'Europe', 'Account', 'AG2002', '2024-11-17'),
    (1018, 'Customer reported that their streaming quality drops to low resolution every evening. Identified bandwidth throttling during peak hours and upgraded their QoS priority settings.', 'North America', 'Internet', 'AG2003', '2024-11-18');
```

**Explain**: We're loading 18 realistic customer support transcripts covering different regions (North America, Europe, Asia), product categories (Internet, Billing, Account, Hardware), and issue types. This gives us a rich dataset to search through.

### Step 1.5: Enable Change Tracking

```sql
-- Enable change tracking so the search service can refresh incrementally
ALTER TABLE support_transcripts SET CHANGE_TRACKING = TRUE;
```

**Explain**: Change tracking lets Cortex Search detect which rows have been added, updated, or deleted since the last index refresh. This enables efficient incremental refreshes instead of rebuilding the entire index each time.

### Step 1.6: Preview the Data

```sql
-- See what we're working with
SELECT * FROM support_transcripts LIMIT 5;
```

**Explain**: Let's look at the data. The `transcript_text` column contains the unstructured support conversations - this is what we'll make searchable with Cortex Search.

```sql
-- Count total transcripts and see the distribution
SELECT COUNT(*) AS total_transcripts FROM support_transcripts;
```

```sql
-- Distribution by region
SELECT region, COUNT(*) AS count
FROM support_transcripts
GROUP BY region
ORDER BY count DESC;
```

```sql
-- Distribution by product category
SELECT product_category, COUNT(*) AS count
FROM support_transcripts
GROUP BY product_category
ORDER BY count DESC;
```

---

## Lesson 2: Create a Cortex Search Service

**Learning Objective**: Create a Cortex Search Service that builds a hybrid (vector + keyword) search index over the transcript data.

### Step 2.1: Understand What We're Building

Before writing any code, explain:

> "A Cortex Search Service is a managed search index that Snowflake builds and maintains for you. When we create one, Snowflake will:
> 1. Generate vector embeddings for each transcript using an embedding model
> 2. Build a keyword index for exact term matching
> 3. Combine both into a hybrid search engine
> 4. Automatically refresh the index based on our TARGET_LAG setting
>
> We don't need to manage embeddings, indexes, or infrastructure - Snowflake handles all of it."

### Step 2.2: Create the Cortex Search Service

```sql
-- Create the Cortex Search Service
CREATE OR REPLACE CORTEX SEARCH SERVICE transcript_search_service
  ON transcript_text
  ATTRIBUTES region, product_category
  WAREHOUSE = SNOWFLAKE_LEARNING_WH
  TARGET_LAG = '1 hour'
  EMBEDDING_MODEL = 'snowflake-arctic-embed-l-v2.0'
  AS (
    SELECT
        transcript_id,
        transcript_text,
        region,
        product_category,
        agent_id,
        created_date
    FROM support_transcripts
  );
```

**Explain**: Let's break down each part:
- **ON transcript_text**: This is the column that will be searched. Cortex Search will build embeddings and keyword indexes on this column.
- **ATTRIBUTES region, product_category**: These columns can be used for filtering search results. For example, we can search only within a specific region.
- **WAREHOUSE**: The warehouse used to build and refresh the index.
- **TARGET_LAG = '1 hour'**: The index should stay within 1 hour of the source data. If new transcripts are added, the index will refresh within an hour.
- **EMBEDDING_MODEL**: The model used to generate vector embeddings. `snowflake-arctic-embed-l-v2.0` is Snowflake's high-quality embedding model.
- **AS (SELECT ...)**: The source query that defines what data goes into the index. All selected columns are returned in search results.

**Note**: Adjust the WAREHOUSE name to match your environment (e.g., `COMPUTE_WH` if not using SNOWFLAKE_LEARNING).

### Step 2.3: Verify the Service

```sql
-- Check that the service was created
SHOW CORTEX SEARCH SERVICES;
```

**Explain**: This shows all Cortex Search Services in the current schema. You should see `TRANSCRIPT_SEARCH_SERVICE` listed with its creation time and status.

### Step 2.4: Inspect the Indexed Data

```sql
-- Scan the data that was indexed
SELECT * FROM TABLE(
    CORTEX_SEARCH_DATA_SCAN(
        SERVICE_NAME => 'transcript_search_service'
    )
) LIMIT 5;
```

**Explain**: `CORTEX_SEARCH_DATA_SCAN` lets us peek inside the search index to confirm that our data was indexed correctly. You should see the transcript text, region, product category, and other columns we selected.

---

## Lesson 3: Query the Service

**Learning Objective**: Query the Cortex Search Service using SQL (SEARCH_PREVIEW) and the Python SDK, including filtering results.

### Step 3.1: Basic Search with SEARCH_PREVIEW

```sql
-- Search for transcripts about internet connectivity
SELECT PARSE_JSON(
    SNOWFLAKE.CORTEX.SEARCH_PREVIEW(
        'transcript_search_service',
        '{
            "query": "internet connection problems",
            "columns": ["transcript_text", "region", "product_category"],
            "limit": 3
        }'
    )
)['results'] AS results;
```

**Explain**: `SEARCH_PREVIEW` lets us test our search service directly in SQL. We pass a JSON object with:
- **query**: The search text. Cortex Search finds transcripts that are semantically similar to this, not just exact matches.
- **columns**: Which columns to return in the results.
- **limit**: Maximum number of results (up to 1000).

Notice the results include transcripts about Wi-Fi issues, slow speeds, and connectivity drops - even though our query said "internet connection problems" and the transcripts use different words. That's semantic search in action.

### Step 3.2: Search with a Filter

```sql
-- Search only within North America
SELECT PARSE_JSON(
    SNOWFLAKE.CORTEX.SEARCH_PREVIEW(
        'transcript_search_service',
        '{
            "query": "billing dispute refund",
            "columns": ["transcript_text", "region", "product_category"],
            "filter": {"@eq": {"region": "North America"}},
            "limit": 5
        }'
    )
)['results'] AS results;
```

**Explain**: The `filter` parameter narrows results to only transcripts from North America. The `@eq` operator does exact matching on the `region` column. We can filter on any column listed in the `ATTRIBUTES` clause when we created the service.

### Step 3.3: Search with Combined Filters

```sql
-- Search for billing issues in Europe
SELECT PARSE_JSON(
    SNOWFLAKE.CORTEX.SEARCH_PREVIEW(
        'transcript_search_service',
        '{
            "query": "how much does the service cost",
            "columns": ["transcript_text", "region", "product_category"],
            "filter": {
                "@and": [
                    {"@eq": {"region": "Europe"}},
                    {"@eq": {"product_category": "Billing"}}
                ]
            },
            "limit": 5
        }'
    )
)['results'] AS results;
```

**Explain**: We can combine multiple filters using `@and`. This query searches for cost-related transcripts that are both in Europe AND in the Billing category. You can also use `@or` and `@not` for more complex filter logic.

### Step 3.4: Query with the Python SDK

```python
# Import required packages
from snowflake.core import Root
from snowflake.snowpark.context import get_active_session

# Get the active session
session = get_active_session()
root = Root(session)
```

**Explain**: The Python SDK gives us a programmatic way to query Cortex Search. We start with a Snowpark session and create a Root object, which is the entry point for Snowflake's Python APIs.

```python
# Navigate to the search service
# Replace <DB> and <SCHEMA> with your actual database and schema names
search_service = (
    root.databases[session.get_current_database()]
    .schemas[session.get_current_schema()]
    .cortex_search_services["TRANSCRIPT_SEARCH_SERVICE"]
)

# Search for password reset issues
resp = search_service.search(
    query="customer locked out of account",
    columns=["transcript_text", "region", "product_category", "agent_id"],
    filter={"@eq": {"product_category": "Account"}},
    limit=3
)

print(resp.to_json())
```

**Explain**: The Python API mirrors what we did in SQL:
- `.search()` takes the same parameters: query, columns, filter, and limit.
- `resp.to_json()` returns the results as a JSON string.
- This is the same API you'd use in a Streamlit app, a Python script, or any application.

### Step 3.5: Explore the Results

```python
import json

# Parse the results
results = json.loads(resp.to_json())

# Print each result in a readable format
for i, result in enumerate(results['results'], 1):
    print(f"\n--- Result {i} ---")
    print(f"Region: {result['region']}")
    print(f"Category: {result['product_category']}")
    print(f"Agent: {result['agent_id']}")
    print(f"Transcript: {result['transcript_text'][:100]}...")
```

**Explain**: The results are ranked by relevance. The top result is the most semantically similar to our query "customer locked out of account". Notice how Cortex Search finds relevant transcripts even when the exact words don't match - it understands the meaning behind the query.

---

## Lesson 4: Build a RAG Chatbot

**Learning Objective**: Combine Cortex Search with Cortex COMPLETE to build a Retrieval Augmented Generation (RAG) chatbot that answers questions grounded in support transcript data.

### Step 4.1: Understand RAG

Before writing code, explain:

> "RAG (Retrieval Augmented Generation) is a pattern that combines two steps:
> 1. **Retrieve**: Search your data for relevant context (this is what Cortex Search does)
> 2. **Generate**: Pass that context to an LLM, which produces a natural language answer grounded in your data
>
> Without RAG, an LLM can only answer from its training data. With RAG, it answers using YOUR data - your support transcripts, your documentation, your knowledge base. This means the answers are accurate, up-to-date, and specific to your business."

### Step 4.2: RAG in SQL

```sql
-- Step 1: Search for relevant transcripts
-- Step 2: Use them as context for the LLM
WITH search_results AS (
    SELECT PARSE_JSON(
        SNOWFLAKE.CORTEX.SEARCH_PREVIEW(
            'transcript_search_service',
            '{
                "query": "How do I reset my password?",
                "columns": ["transcript_text"],
                "limit": 3
            }'
        )
    )['results'] AS results
),
context AS (
    SELECT ARRAY_TO_STRING(
        ARRAY_AGG(r.value:transcript_text::STRING),
        '\n---\n'
    ) AS context_text
    FROM search_results,
         LATERAL FLATTEN(input => results) r
)
SELECT SNOWFLAKE.CORTEX.COMPLETE(
    'mistral-large2',
    CONCAT(
        'You are a helpful customer support assistant for Tasty Bytes. ',
        'Based on the following past support conversations, answer the customer question. ',
        'If the answer is not found in the conversations, say so.\n\n',
        'Past support conversations:\n', context_text, '\n\n',
        'Customer question: How do I reset my password?\n\n',
        'Answer:'
    )
) AS ai_response
FROM context;
```

**Explain**: This is the complete RAG pattern in a single SQL query:
1. **search_results CTE**: Uses SEARCH_PREVIEW to find the 3 most relevant transcripts about password resets.
2. **context CTE**: Flattens the JSON results and concatenates the transcript texts into a single context string.
3. **Final SELECT**: Passes the context to `CORTEX.COMPLETE` with a prompt that instructs the LLM to answer using ONLY the provided transcripts.

The LLM's response will be grounded in actual support conversations, not made up.

### Step 4.3: RAG in Python

```python
import json
from snowflake.core import Root
from snowflake.snowpark.context import get_active_session

session = get_active_session()
root = Root(session)

def ask_support_bot(question, region_filter=None):
    """Search transcripts and generate an answer using RAG."""
    
    # Step 1: Retrieve relevant transcripts
    search_service = (
        root.databases[session.get_current_database()]
        .schemas[session.get_current_schema()]
        .cortex_search_services["TRANSCRIPT_SEARCH_SERVICE"]
    )
    
    search_params = {
        "query": question,
        "columns": ["transcript_text", "region", "product_category"],
        "limit": 3
    }
    
    if region_filter:
        search_params["filter"] = {"@eq": {"region": region_filter}}
    
    resp = search_service.search(**search_params)
    results = json.loads(resp.to_json())
    
    # Step 2: Build context from search results
    context_parts = []
    for r in results['results']:
        context_parts.append(
            f"[{r['region']} - {r['product_category']}] {r['transcript_text']}"
        )
    context = "\n---\n".join(context_parts)
    
    # Step 3: Generate answer with CORTEX.COMPLETE
    prompt = f"""You are a helpful customer support assistant for Tasty Bytes.
Based on the following past support conversations, answer the customer question.
If the answer is not found in the conversations, say so.

Past support conversations:
{context}

Customer question: {question}

Answer:"""
    
    response = session.sql(
        "SELECT SNOWFLAKE.CORTEX.COMPLETE('mistral-large2', ?) AS response",
        params=[prompt]
    ).collect()
    
    return response[0]['RESPONSE']
```

**Explain**: This Python function wraps the RAG pattern into a reusable `ask_support_bot` function:
1. It searches for relevant transcripts using Cortex Search (with optional region filtering).
2. It builds a context string from the search results, including region and category metadata.
3. It generates an answer using `CORTEX.COMPLETE` with a well-structured prompt.

### Step 4.4: Try Different Questions

```python
# Question 1: Password reset
print("Q: How do I reset my password?")
print("A:", ask_support_bot("How do I reset my password?"))
print()

# Question 2: Billing with region filter
print("Q: How can I get a refund? (North America only)")
print("A:", ask_support_bot("How can I get a refund?", region_filter="North America"))
print()

# Question 3: Technical issue
print("Q: My internet keeps disconnecting, what should I do?")
print("A:", ask_support_bot("My internet keeps disconnecting, what should I do?"))
```

**Explain**: Try asking different questions and notice how the RAG chatbot:
- Finds relevant past conversations using semantic search
- Generates answers based on actual support resolutions
- Can be filtered by region for location-specific answers
- Admits when it doesn't have relevant information

---

## Final Verification

```sql
-- Verify the search service exists and is active
SHOW CORTEX SEARCH SERVICES;
```

```sql
-- Verify search works
SELECT PARSE_JSON(
    SNOWFLAKE.CORTEX.SEARCH_PREVIEW(
        'transcript_search_service',
        '{
            "query": "help with my account",
            "columns": ["transcript_text", "region"],
            "limit": 3
        }'
    )
)['results'] AS results;
```

---

## Summary

| What You Learned | How |
|------------------|-----|
| Create a search service | `CREATE CORTEX SEARCH SERVICE ... ON column ATTRIBUTES ... TARGET_LAG ...` |
| Enable incremental refresh | `ALTER TABLE ... SET CHANGE_TRACKING = TRUE` |
| Search in SQL | `SNOWFLAKE.CORTEX.SEARCH_PREVIEW(service, json_query)` |
| Filter search results | `"filter": {"@eq": {"column": "value"}}` |
| Search in Python | `root.databases[...].cortex_search_services[...].search(...)` |
| Build a RAG chatbot | Search -> Build context -> `CORTEX.COMPLETE(model, prompt)` |
