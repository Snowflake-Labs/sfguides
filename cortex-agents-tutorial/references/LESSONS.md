# Cortex Agents Tutorial - Lesson SQL Reference

This file contains all SQL and Python code for the tutorial, organized by section. Execute each section step-by-step, explaining as you go.

**IMPORTANT**: This file contains TESTED, WORKING syntax as of March 2026.
**NOTE**: `CREATE AGENT ... FROM SPECIFICATION $$yaml$$` supports full tool configuration inline, including `tools`, `tool_resources`, and `execution_environment`.

---

## Section 1: Setup & Prerequisites

**Learning Objective**: Set up the environment and verify Cortex Agent access.

### Step 1.1: Set Up Environment

```sql
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;

CREATE DATABASE IF NOT EXISTS CORTEX_AGENTS_LAB;
USE DATABASE CORTEX_AGENTS_LAB;

CREATE SCHEMA IF NOT EXISTS TUTORIAL;
USE SCHEMA TUTORIAL;
```

### Step 1.2: Verify Cortex Access

```sql
SELECT SNOWFLAKE.CORTEX.COMPLETE('claude-3-5-sonnet', 'Say hello in one word');
```

**Explain**: This quick test confirms your account has Cortex AI enabled.

---

## Section 2: Create Sample Data

**Learning Objective**: Load structured and unstructured data that the agent will query.

### Step 2.1: Create Structured Data (Sales)

```sql
CREATE OR REPLACE TABLE sales (
    sale_id NUMBER AUTOINCREMENT,
    sale_date DATE,
    product_name VARCHAR,
    category VARCHAR,
    region VARCHAR,
    quantity NUMBER,
    unit_price NUMBER(10,2),
    total_amount NUMBER(10,2),
    customer_segment VARCHAR
);
```

### Step 2.2: Load Sample Sales Data

```sql
INSERT INTO sales (sale_date, product_name, category, region, quantity, unit_price, total_amount, customer_segment)
VALUES
    ('2024-01-15', 'Laptop Pro', 'Electronics', 'North America', 10, 1299.99, 12999.90, 'Enterprise'),
    ('2024-01-16', 'Wireless Mouse', 'Electronics', 'Europe', 50, 29.99, 1499.50, 'SMB'),
    ('2024-01-17', 'Office Chair', 'Furniture', 'North America', 20, 299.99, 5999.80, 'Enterprise'),
    ('2024-01-18', 'Standing Desk', 'Furniture', 'Asia Pacific', 15, 499.99, 7499.85, 'SMB'),
    ('2024-01-19', 'Monitor 27"', 'Electronics', 'Europe', 30, 399.99, 11999.70, 'Enterprise'),
    ('2024-01-20', 'Keyboard Pro', 'Electronics', 'North America', 100, 149.99, 14999.00, 'Consumer'),
    ('2024-02-01', 'Laptop Pro', 'Electronics', 'Asia Pacific', 25, 1299.99, 32499.75, 'Enterprise'),
    ('2024-02-05', 'Webcam HD', 'Electronics', 'North America', 75, 79.99, 5999.25, 'SMB'),
    ('2024-02-10', 'Office Chair', 'Furniture', 'Europe', 40, 299.99, 11999.60, 'Enterprise'),
    ('2024-02-15', 'Headphones', 'Electronics', 'North America', 60, 199.99, 11999.40, 'Consumer'),
    ('2024-03-01', 'Standing Desk', 'Furniture', 'North America', 35, 499.99, 17499.65, 'Enterprise'),
    ('2024-03-10', 'Laptop Pro', 'Electronics', 'Europe', 20, 1299.99, 25999.80, 'SMB');
```

### Step 2.3: Create Inventory Table

```sql
CREATE OR REPLACE TABLE inventory (
    product_name VARCHAR,
    sku VARCHAR,
    quantity_in_stock NUMBER,
    reorder_level NUMBER,
    unit_cost NUMBER(10,2),
    last_restocked DATE
);

INSERT INTO inventory VALUES
    ('Laptop Pro', 'LP-001', 45, 20, 899.99, '2024-03-01'),
    ('Wireless Mouse', 'WM-002', 500, 100, 12.99, '2024-02-15'),
    ('Office Chair', 'OC-003', 75, 25, 149.99, '2024-02-20'),
    ('Standing Desk', 'SD-004', 30, 15, 299.99, '2024-03-05'),
    ('Monitor 27"', 'MN-005', 60, 20, 249.99, '2024-02-28'),
    ('Keyboard Pro', 'KP-006', 200, 50, 79.99, '2024-03-10');
```

### Step 2.4: Create Product Documentation Table

```sql
CREATE OR REPLACE TABLE product_docs (
    doc_id NUMBER AUTOINCREMENT,
    product_name VARCHAR,
    doc_type VARCHAR,
    content VARCHAR,
    last_updated DATE
);

INSERT INTO product_docs (product_name, doc_type, content, last_updated)
VALUES
    ('Laptop Pro', 'specifications', 
     'The Laptop Pro features a 15.6-inch 4K display, Intel i9 processor, 32GB RAM, and 1TB SSD. Battery life is up to 12 hours. Includes Thunderbolt 4 ports and Wi-Fi 6E. Weight: 4.2 lbs. Warranty: 3 years standard, extendable to 5 years.',
     '2024-01-01'),
    ('Laptop Pro', 'troubleshooting',
     'Common issues: 1) Battery drain - check background apps and reduce screen brightness. 2) Overheating - ensure vents are not blocked, use on hard surface. 3) Slow performance - check for updates, run disk cleanup. 4) Wi-Fi issues - update network drivers, reset network settings.',
     '2024-01-15'),
    ('Standing Desk', 'specifications',
     'Electric standing desk with memory presets. Height range: 28-48 inches. Desktop size: 60x30 inches. Weight capacity: 300 lbs. Motor: dual motor system for stability. Includes cable management tray and anti-collision sensor.',
     '2024-01-01'),
    ('Standing Desk', 'assembly',
     'Assembly instructions: 1) Attach legs to frame using provided bolts. 2) Connect motor cables to control box. 3) Mount desktop to frame. 4) Connect power cord. 5) Program height presets using control panel. Assembly time: approximately 45 minutes. Tools needed: Phillips screwdriver.',
     '2024-01-10'),
    ('Office Chair', 'specifications',
     'Ergonomic office chair with lumbar support. Adjustable armrests, seat height, and tilt tension. Breathable mesh back. Seat dimensions: 20x19 inches. Weight capacity: 275 lbs. Warranty: 5 years on frame, 2 years on upholstery.',
     '2024-01-01'),
    ('Office Chair', 'care',
     'Care instructions: Clean mesh with mild soap and water. Lubricate casters annually. Check and tighten bolts every 6 months. Do not exceed weight capacity. Store in dry environment. Replace gas cylinder if chair sinks.',
     '2024-02-01');
```

### Step 2.5: Verify All Data

```sql
SELECT 'sales' as table_name, COUNT(*) as row_count FROM sales
UNION ALL SELECT 'inventory', COUNT(*) FROM inventory
UNION ALL SELECT 'product_docs', COUNT(*) FROM product_docs;
```

---

## Section 3: Create Semantic View for Cortex Analyst

**Learning Objective**: Build a semantic model that enables natural language queries against structured data.

### CRITICAL: Semantic Views use SQL clauses, NOT embedded YAML!

### Step 3.1: Create the Semantic View

```sql
CREATE OR REPLACE SEMANTIC VIEW sales_semantic_view
  TABLES (
    sales AS CORTEX_AGENTS_LAB.TUTORIAL.SALES
      PRIMARY KEY (sale_id)
      COMMENT = 'Transaction-level sales data'
  )
  DIMENSIONS (
    sales.product_name AS product_name
      COMMENT = 'Name of the product sold',
    sales.category AS category
      COMMENT = 'Product category (Electronics, Furniture)',
    sales.region AS region
      COMMENT = 'Sales region (North America, Europe, Asia Pacific)',
    sales.customer_segment AS customer_segment
      COMMENT = 'Customer type (Enterprise, SMB, Consumer)'
  )
  METRICS (
    sales.total_sales AS SUM(total_amount)
      COMMENT = 'Total sales amount in USD',
    sales.total_quantity AS SUM(quantity)
      COMMENT = 'Total units sold',
    sales.transaction_count AS COUNT(*)
      COMMENT = 'Number of sales transactions'
  )
  COMMENT = 'Sales data for analyzing revenue, products, and regional performance';
```

### Step 3.2: Verify

```sql
SHOW SEMANTIC VIEWS;
```

### Step 3.3: Test the Semantic View

```sql
SELECT * FROM SEMANTIC_VIEW(
    CORTEX_AGENTS_LAB.TUTORIAL.SALES_SEMANTIC_VIEW
    METRICS total_sales, total_quantity
    DIMENSIONS region
)
ORDER BY total_sales DESC;
```

### Step 3.4: Quick Cortex Analyst Demo

Test Cortex Analyst independently before wiring it into the agent:

```sql
SELECT SNOWFLAKE.CORTEX.COMPLETE(
    'claude-4-sonnet',
    'Given a sales table with columns: product_name, category, region, quantity, unit_price, total_amount, customer_segment - write a SQL query to find total sales by region. Return ONLY the SQL.'
);
```

---

## Section 4: Create Cortex Search Service

**Learning Objective**: Index unstructured documents for semantic search (RAG).

### Step 4.1: Create the Search Service

```sql
CREATE OR REPLACE CORTEX SEARCH SERVICE product_search_service
  ON content
  ATTRIBUTES product_name, doc_type
  WAREHOUSE = COMPUTE_WH
  TARGET_LAG = '1 hour'
  AS (
    SELECT 
      doc_id,
      product_name,
      doc_type,
      content
    FROM product_docs
  );
```

### Step 4.2: Verify

```sql
SHOW CORTEX SEARCH SERVICES;
```

### Step 4.3: Test Search

```sql
SELECT 
  r.value:product_name::STRING AS product_name,
  r.value:content::STRING AS content
FROM TABLE(FLATTEN(
  PARSE_JSON(
    SNOWFLAKE.CORTEX.SEARCH_PREVIEW(
      'CORTEX_AGENTS_LAB.TUTORIAL.PRODUCT_SEARCH_SERVICE',
      '{"query": "laptop overheating fix", "columns": ["product_name", "content"], "limit": 3}'
    )
  )['results']
)) r;
```

### Step 4.4: Quick RAG Demo

Show how Cortex Search results feed into an LLM for a complete RAG pipeline:

```sql
SELECT SNOWFLAKE.CORTEX.COMPLETE(
    'claude-4-sonnet',
    'Based on this documentation, how do I fix laptop overheating? Documentation: ' ||
    (SELECT LISTAGG(r.value:content::STRING, ' | ') 
     FROM TABLE(FLATTEN(
       PARSE_JSON(
         SNOWFLAKE.CORTEX.SEARCH_PREVIEW(
           'CORTEX_AGENTS_LAB.TUTORIAL.PRODUCT_SEARCH_SERVICE',
           '{"query": "laptop overheating", "columns": ["content"], "limit": 2}'
         )
       )['results']
     )) r)
);
```

---

## Section 5: Build the Cortex Agent

**Learning Objective**: Create a Cortex Agent with full tool configuration via SQL and test it.

### Step 5.1: Create the Agent

```sql
CREATE OR REPLACE AGENT tutorial_agent
  COMMENT = 'Tutorial agent that routes questions to Analyst (structured) or Search (unstructured)'
  FROM SPECIFICATION $$
models:
  orchestration: claude-4-sonnet
orchestration:
  budget:
    seconds: 30
    tokens: 16000
instructions:
  system: "You are a helpful assistant for a retail business. You can answer questions about sales data and product documentation."
  orchestration: "Use Analyst for any question about sales, revenue, quantities, or metrics. Use Search for product documentation, troubleshooting, or how-to questions."
  response: "Be concise and include relevant numbers or details from the tools."
tools:
  - tool_spec:
      type: "cortex_analyst_text_to_sql"
      name: "Analyst"
      description: "Queries structured sales data by converting natural language to SQL"
  - tool_spec:
      type: "cortex_search"
      name: "Search"
      description: "Searches product documentation and troubleshooting guides"
tool_resources:
  Analyst:
    semantic_view: "CORTEX_AGENTS_LAB.TUTORIAL.SALES_SEMANTIC_VIEW"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
  Search:
    name: "CORTEX_AGENTS_LAB.TUTORIAL.PRODUCT_SEARCH_SERVICE"
    max_results: "3"
$$;
```

**CRITICAL**: The `execution_environment` block under `Analyst` is required. Without it (or using a bare `warehouse:` key), the agent will create successfully but `DATA_AGENT_RUN` will fail with error 399504: "The Analyst tool is missing an execution environment."

### Step 5.2: Verify

```sql
SHOW AGENTS;
```

### Step 5.3: Test Agent - Structured Data Question (Python)

```python
import json
from snowflake.snowpark.context import get_active_session

session = get_active_session()

# Run the agent with a structured data question
result = session.sql("""
  SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'CORTEX_AGENTS_LAB.TUTORIAL.TUTORIAL_AGENT',
    $${"messages": [{"role": "user", "content": [{"type": "text", "text": "What are total sales by region?"}]}]}$$
  ) AS resp
""").collect()

resp = json.loads(result[0]["RESP"])

# Parse and display each content item
for item in resp.get("content", []):
    item_type = item.get("type")

    if item_type == "thinking":
        print("=== THINKING ===")
        print(item["thinking"]["text"])
        print()

    elif item_type == "tool_use":
        print(f"=== TOOL CALL: {item['tool_use'].get('name', '')} ({item['tool_use'].get('type', '')}) ===")
        print()

    elif item_type == "tool_result":
        tr = item["tool_result"]
        print(f"=== TOOL RESULT: {tr.get('name', '')} ===")
        content_json = tr.get("content", [{}])[0].get("json", {})
        if content_json.get("sql"):
            print(f"Generated SQL:\n{content_json['sql']}\n")
        if content_json.get("sql_explanation"):
            print(f"Explanation: {content_json['sql_explanation']}\n")
        if content_json.get("result_set", {}).get("data"):
            meta = content_json["result_set"].get("resultSetMetaData", {})
            cols = [col["name"] for col in meta.get("rowType", [])]
            data = content_json["result_set"]["data"]
            if cols:
                print(" | ".join(cols))
                print("-" * (len(" | ".join(cols))))
            for row in data:
                print(" | ".join(str(v) for v in row))
            print()

    elif item_type == "text":
        print("=== ANSWER ===")
        print(item["text"])
        print()
```

**Explain**: This Python parsing extracts the full agent response including thinking, the generated SQL query, result data, and the final answer. Raw SQL output would truncate or hide most of this.

### Step 5.4: Test Agent - Documentation Question (Python)

```python
# Run the agent with a documentation question
result = session.sql("""
  SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'CORTEX_AGENTS_LAB.TUTORIAL.TUTORIAL_AGENT',
    $${"messages": [{"role": "user", "content": [{"type": "text", "text": "How do I fix laptop overheating?"}]}]}$$
  ) AS resp
""").collect()

resp = json.loads(result[0]["RESP"])

for item in resp.get("content", []):
    item_type = item.get("type")

    if item_type == "thinking":
        print("=== THINKING ===")
        print(item["thinking"]["text"])
        print()

    elif item_type == "tool_use":
        print(f"=== TOOL CALL: {item['tool_use'].get('name', '')} ({item['tool_use'].get('type', '')}) ===")
        print()

    elif item_type == "tool_result":
        tr = item["tool_result"]
        print(f"=== TOOL RESULT: {tr.get('name', '')} (status: {tr.get('status', '')}) ===")
        print()

    elif item_type == "text":
        print("=== ANSWER ===")
        print(item["text"])
        print()
```

---

## Section 6: Use with Snowflake Intelligence

**Learning Objective**: Interact with the agent via Snowflake Intelligence chat UI.

Now that you've created the `tutorial_agent` object, you can chat with it directly in **Snowflake Intelligence** -- no extra code needed.

**To try it:**
1. In Snowsight, go to **AI & ML > Snowflake Intelligence**
2. Select `TUTORIAL_AGENT` from the agent picker in the chat bar
3. Ask any question -- the agent routes to Analyst or Search automatically
   - Try **"What are total sales by region?"** (routes to Cortex Analyst)
   - Try **"How do I assemble the standing desk?"** (routes to Cortex Search)

Snowflake Intelligence gives you a built-in chat interface with:
- **Streaming responses** -- see the agent's thinking and tool calls in real time
- **Multi-turn conversations** -- ask follow-up questions that build on previous context
- **Tool call visibility** -- see which tool the agent chose and the generated SQL or search results

This is the fastest way to interact with the agent you just built. For a custom UI, you can build a Streamlit app on top of the same agent using the `DATA_AGENT_RUN` SQL function or the Cortex Agents REST API.

---

## Summary

What you built in this tutorial:

| Component | What It Does |
|-----------|-------------|
| **Semantic View** | Defines the schema for natural language to SQL translation |
| **Cortex Search Service** | Indexes product docs for semantic retrieval |
| **Cortex Agent** | Orchestrates between Analyst and Search tools |
| **Python Parsing** | Extracts thinking, SQL, results, and answers from agent responses |
| **Snowflake Intelligence** | Chat UI for interacting with the agent |

---

## Cleanup (Only if requested)

```sql
DROP AGENT IF EXISTS tutorial_agent;
DROP CORTEX SEARCH SERVICE IF EXISTS product_search_service;
DROP SEMANTIC VIEW IF EXISTS sales_semantic_view;
DROP TABLE IF EXISTS sales;
DROP TABLE IF EXISTS inventory;
DROP TABLE IF EXISTS product_docs;
-- DROP SCHEMA IF EXISTS TUTORIAL;
-- DROP DATABASE IF EXISTS CORTEX_AGENTS_LAB;
```

---

## Best Practices Summary

| Practice | Implementation |
|----------|---------------|
| **Descriptive Tool Names** | `Analyst`, `Search` -- clear purpose |
| **Tool Descriptions** | Include what data the tool accesses and when to use it |
| **Instructions** | `system` + `orchestration` + `response` keys |
| **execution_environment** | Required for Analyst: `{type: warehouse, warehouse: WH}` |
| **Python Parsing** | Use `json.loads()` to extract full response details |

## Tool Type Reference

| Type | Purpose | Example |
|------|---------|---------|
| `cortex_analyst_text_to_sql` | SQL on structured data | Analyst |
| `cortex_search` | Search unstructured docs | Search |
| `generic` | Custom procedures | InventoryLookup |
| `web_search` | External WWW search | WebSearch |
| `data_to_chart` | Auto-generate visualizations | DataToChart |
