# Cortex Agents with Custom Tools Tutorial - Lesson SQL Reference

This file contains all SQL and Python code for the tutorial, organized by section. Execute each section step-by-step, explaining as you go.

**IMPORTANT**: This file contains TESTED, WORKING syntax as of March 2026.
**NOTE**: `CREATE AGENT ... FROM SPECIFICATION $$yaml$$` supports full tool configuration inline, including `tools`, `tool_resources`, `execution_environment`, and custom tool `input_schema`.

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

---

## Section 5: Create Custom Tool Procedures

**Learning Objective**: Build stored procedures that the agent can call as custom tools for inventory checks, price calculations, and product summaries.

### Why Stored Procedures?

Custom agent tools MUST be stored procedures (`CREATE PROCEDURE`), NOT UDFs (`CREATE FUNCTION`). Key requirements:
- `EXECUTE AS CALLER` for proper privilege flow
- `p_` prefixed parameters to avoid column name collisions
- Colon prefix (`:p_param`) when referencing parameters inside SQL
- `DECLARE`/`BEGIN`/`END` block with `$$` delimiters

### Step 5.1: Create Inventory Lookup Procedure

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
    SELECT TO_VARCHAR(quantity_in_stock) || ' units in stock (reorder level: ' || TO_VARCHAR(reorder_level) || ')'
      INTO :result
    FROM CORTEX_AGENTS_LAB.TUTORIAL.INVENTORY
    WHERE UPPER(product_name) = UPPER(:p_product_name);
    RETURN result;
END;
$$;
```

### Step 5.2: Create Price Calculator Procedure

```sql
CREATE OR REPLACE PROCEDURE calculate_price_proc(p_product_name VARCHAR, p_quantity NUMBER)
RETURNS VARCHAR
LANGUAGE SQL
EXECUTE AS CALLER
AS
$$
DECLARE
    result VARCHAR;
BEGIN
    SELECT 
        'Unit price: $' || TO_VARCHAR(unit_price, '999,999.99') ||
        ' | Quantity: ' || TO_VARCHAR(:p_quantity) ||
        ' | Discount: ' || TO_VARCHAR(
            CASE 
                WHEN :p_quantity >= 100 THEN 15
                WHEN :p_quantity >= 50 THEN 10
                WHEN :p_quantity >= 10 THEN 5
                ELSE 0
            END
        ) || '%' ||
        ' | Total: $' || TO_VARCHAR(
            unit_price * :p_quantity * (1 - 
                CASE 
                    WHEN :p_quantity >= 100 THEN 0.15
                    WHEN :p_quantity >= 50 THEN 0.10
                    WHEN :p_quantity >= 10 THEN 0.05
                    ELSE 0
                END
            ), '999,999.99')
      INTO :result
    FROM CORTEX_AGENTS_LAB.TUTORIAL.SALES
    WHERE UPPER(product_name) = UPPER(:p_product_name)
    LIMIT 1;
    RETURN result;
END;
$$;
```

### Step 5.3: Create Product Summary Procedure

```sql
CREATE OR REPLACE PROCEDURE get_product_summary_proc(p_product_name VARCHAR)
RETURNS VARCHAR
LANGUAGE SQL
EXECUTE AS CALLER
AS
$$
DECLARE
    result VARCHAR;
BEGIN
    SELECT 
        'Product: ' || :p_product_name || 
        ' | Total Revenue: $' || TO_VARCHAR(SUM(total_amount), '999,999.99') ||
        ' | Units Sold: ' || TO_VARCHAR(SUM(quantity)) ||
        ' | Avg Price: $' || TO_VARCHAR(AVG(unit_price), '999.99')
      INTO :result
    FROM CORTEX_AGENTS_LAB.TUTORIAL.SALES
    WHERE UPPER(product_name) = UPPER(:p_product_name);
    RETURN result;
END;
$$;
```

### Step 5.4: Verify Procedures Work

Before wiring into the agent, confirm each procedure returns correct results:

```sql
CALL check_inventory_proc('Standing Desk');
CALL calculate_price_proc('Laptop Pro', 50);
CALL get_product_summary_proc('Laptop Pro');
```

---

## Section 6: Build the 5-Tool Agent

**Learning Objective**: Create a Cortex Agent with all 5 tools configured via SQL.

### Step 6.1: Create the Agent

```sql
CREATE OR REPLACE AGENT sales_assistant
  COMMENT = 'AI assistant with 5 tools: Analyst, Search, InventoryLookup, PriceCalculator, ProductSummary'
  FROM SPECIFICATION $$
models:
  orchestration: claude-4-sonnet
orchestration:
  budget:
    seconds: 30
    tokens: 16000
instructions:
  system: |
    You are Sales Assistant, an AI-powered sales intelligence assistant for a retail business.
    You can answer questions about sales data, product documentation, inventory levels, pricing, and product performance.
  orchestration: |
    For sales metrics, revenue, trends, or regional comparisons: Use Analyst
    For product documentation, troubleshooting, or how-to questions: Use Search
    For stock levels or inventory availability: Use InventoryLookup
    For pricing quotes or bulk discount calculations: Use PriceCalculator
    For quick product performance overviews: Use ProductSummary
    If a question spans multiple areas, use the most relevant tools and combine results.
  response: |
    Be concise and professional.
    Lead with the direct answer, then provide supporting details.
    Include relevant numbers with units and currency symbols.
tools:
  - tool_spec:
      type: "cortex_analyst_text_to_sql"
      name: "Analyst"
      description: "Queries structured sales data by converting natural language to SQL. Covers revenue, quantities, products, regions, and customer segments."
  - tool_spec:
      type: "cortex_search"
      name: "Search"
      description: "Searches product documentation for specifications, troubleshooting guides, assembly instructions, and care information."
  - tool_spec:
      type: "generic"
      name: "InventoryLookup"
      description: "Checks current stock levels and reorder status for a specific product."
      input_schema:
        type: "object"
        properties:
          p_product_name:
            type: "string"
            description: "The product name to check inventory for"
        required: ["p_product_name"]
  - tool_spec:
      type: "generic"
      name: "PriceCalculator"
      description: "Calculates pricing with bulk discounts. Discounts: 5% for 10+ units, 10% for 50+, 15% for 100+."
      input_schema:
        type: "object"
        properties:
          p_product_name:
            type: "string"
            description: "The product name to price"
          p_quantity:
            type: "number"
            description: "The quantity to calculate pricing for"
        required: ["p_product_name", "p_quantity"]
  - tool_spec:
      type: "generic"
      name: "ProductSummary"
      description: "Provides a quick overview of a product's sales performance including total revenue, units sold, and average price."
      input_schema:
        type: "object"
        properties:
          p_product_name:
            type: "string"
            description: "The product name to summarize"
        required: ["p_product_name"]
tool_resources:
  Analyst:
    semantic_view: "CORTEX_AGENTS_LAB.TUTORIAL.SALES_SEMANTIC_VIEW"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
  Search:
    name: "CORTEX_AGENTS_LAB.TUTORIAL.PRODUCT_SEARCH_SERVICE"
    max_results: "3"
  InventoryLookup:
    type: "procedure"
    identifier: "CORTEX_AGENTS_LAB.TUTORIAL.CHECK_INVENTORY_PROC"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
  PriceCalculator:
    type: "procedure"
    identifier: "CORTEX_AGENTS_LAB.TUTORIAL.CALCULATE_PRICE_PROC"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
  ProductSummary:
    type: "procedure"
    identifier: "CORTEX_AGENTS_LAB.TUTORIAL.GET_PRODUCT_SUMMARY_PROC"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
$$;
```

**CRITICAL**: The `execution_environment` block is required for BOTH Analyst AND custom (generic) tools. Without it, `DATA_AGENT_RUN` fails with error 399504.

### Step 6.2: Verify

```sql
SHOW AGENTS;
```

---

## Section 7: Test All Tools

**Learning Objective**: Run queries that exercise each tool and observe the agent's orchestration.

### Understanding DATA_AGENT_RUN

```
CREATE AGENT       = Builds the agent (brain + instructions + tools)
DATA_AGENT_RUN     = Sends it a message and gets back the answer
```

You don't tell the agent which tool to use. The agent figures that out from:
1. Your question's intent
2. The orchestration instructions (Section 6)
3. The tool descriptions

### Step 7.1: Test Analyst - Sales Analysis (Python)

```python
import json
from snowflake.snowpark.context import get_active_session

session = get_active_session()

result = session.sql("""
  SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'CORTEX_AGENTS_LAB.TUTORIAL.SALES_ASSISTANT',
    $${"messages": [{"role": "user", "content": [{"type": "text", "text": "What are the total sales by region?"}]}]}$$
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

**Expected**: Agent uses Analyst tool, generates SQL, returns sales by region.

### Step 7.2: Test Search - Documentation (Python)

```python
result = session.sql("""
  SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'CORTEX_AGENTS_LAB.TUTORIAL.SALES_ASSISTANT',
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

**Expected**: Agent uses Search tool, retrieves troubleshooting docs.

### Step 7.3: Test InventoryLookup (Python)

```python
result = session.sql("""
  SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'CORTEX_AGENTS_LAB.TUTORIAL.SALES_ASSISTANT',
    $${"messages": [{"role": "user", "content": [{"type": "text", "text": "How many Standing Desks do we have in stock?"}]}]}$$
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
        print(f"=== TOOL CALL: {item['tool_use'].get('name', '')} ===")
        print()

    elif item_type == "tool_result":
        tr = item["tool_result"]
        print(f"=== TOOL RESULT: {tr.get('name', '')} ===")
        for c in tr.get("content", []):
            if "json" in c:
                print(json.dumps(c["json"], indent=2))
            elif "text" in c:
                print(c["text"])
        print()

    elif item_type == "text":
        print("=== ANSWER ===")
        print(item["text"])
        print()
```

**Expected**: Agent uses InventoryLookup tool, returns "30 units in stock (reorder level: 15)".

### Step 7.4: Test PriceCalculator (Python)

```python
result = session.sql("""
  SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'CORTEX_AGENTS_LAB.TUTORIAL.SALES_ASSISTANT',
    $${"messages": [{"role": "user", "content": [{"type": "text", "text": "What would 50 Laptop Pro units cost with bulk discount?"}]}]}$$
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
        print(f"=== TOOL CALL: {item['tool_use'].get('name', '')} ===")
        print()

    elif item_type == "tool_result":
        tr = item["tool_result"]
        print(f"=== TOOL RESULT: {tr.get('name', '')} ===")
        for c in tr.get("content", []):
            if "json" in c:
                print(json.dumps(c["json"], indent=2))
            elif "text" in c:
                print(c["text"])
        print()

    elif item_type == "text":
        print("=== ANSWER ===")
        print(item["text"])
        print()
```

**Expected**: Agent uses PriceCalculator tool, returns pricing with 10% discount for 50 units.

### Step 7.5: Test ProductSummary (Python)

```python
result = session.sql("""
  SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'CORTEX_AGENTS_LAB.TUTORIAL.SALES_ASSISTANT',
    $${"messages": [{"role": "user", "content": [{"type": "text", "text": "Give me a quick summary of Standing Desk performance"}]}]}$$
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
        print(f"=== TOOL CALL: {item['tool_use'].get('name', '')} ===")
        print()

    elif item_type == "tool_result":
        tr = item["tool_result"]
        print(f"=== TOOL RESULT: {tr.get('name', '')} ===")
        for c in tr.get("content", []):
            if "json" in c:
                print(json.dumps(c["json"], indent=2))
            elif "text" in c:
                print(c["text"])
        print()

    elif item_type == "text":
        print("=== ANSWER ===")
        print(item["text"])
        print()
```

**Expected**: Agent uses ProductSummary tool, returns revenue, units sold, avg price for Standing Desk.

---

## Section 8: Use with Snowflake Intelligence

**Learning Objective**: Interact with the agent via Snowflake Intelligence chat UI.

Now that you've created the `sales_assistant` agent with 5 tools, you can chat with it directly in **Snowflake Intelligence** -- no extra code needed.

**To try it:**
1. In Snowsight, go to **AI & ML > Snowflake Intelligence**
2. Select `SALES_ASSISTANT` from the agent picker in the chat bar
3. Ask any question -- the agent routes to the appropriate tool automatically
   - Try **"What are total sales by region?"** (routes to Cortex Analyst)
   - Try **"How do I assemble the standing desk?"** (routes to Cortex Search)
   - Try **"How many Laptop Pro units are in stock?"** (routes to InventoryLookup)
   - Try **"What would 100 Office Chairs cost?"** (routes to PriceCalculator)
   - Try **"Summarize Laptop Pro performance"** (routes to ProductSummary)

Snowflake Intelligence gives you a built-in chat interface with:
- **Streaming responses** -- see the agent's thinking and tool calls in real time
- **Multi-turn conversations** -- ask follow-up questions that build on previous context
- **Tool call visibility** -- see which tool the agent chose and the results

This is the fastest way to interact with the agent you just built. For a custom UI, you can build a Streamlit app on top of the same agent using the `DATA_AGENT_RUN` SQL function or the Cortex Agents REST API.

---

## Summary

What you built in this tutorial:

| Component | What It Does |
|-----------|-------------|
| **Semantic View** | Defines the schema for natural language to SQL translation |
| **Cortex Search Service** | Indexes product docs for semantic retrieval |
| **3 Stored Procedures** | Custom business logic (inventory, pricing, summaries) |
| **5-Tool Cortex Agent** | Orchestrates across Analyst, Search, and 3 custom tools |
| **Python Parsing** | Extracts thinking, SQL, results, and answers from agent responses |
| **Snowflake Intelligence** | Chat UI for interacting with the agent |

### Tool Routing Summary

| Question Type | Tool Used |
|--------------|-----------|
| Sales metrics, revenue, trends | Analyst (Cortex Analyst) |
| Product documentation, troubleshooting | Search (Cortex Search) |
| Stock levels, inventory | InventoryLookup (Custom) |
| Pricing quotes, bulk discounts | PriceCalculator (Custom) |
| Product performance overview | ProductSummary (Custom) |

---

## Cleanup (Only if requested)

```sql
DROP AGENT IF EXISTS sales_assistant;
DROP CORTEX SEARCH SERVICE IF EXISTS product_search_service;
DROP SEMANTIC VIEW IF EXISTS sales_semantic_view;
DROP PROCEDURE IF EXISTS check_inventory_proc(VARCHAR);
DROP PROCEDURE IF EXISTS calculate_price_proc(VARCHAR, NUMBER);
DROP PROCEDURE IF EXISTS get_product_summary_proc(VARCHAR);
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
| **Descriptive Tool Names** | `Analyst`, `Search`, `InventoryLookup` -- clear purpose |
| **Tool Descriptions** | Include what data the tool accesses and when to use it |
| **Instructions** | `system` + `orchestration` + `response` keys |
| **execution_environment** | Required for Analyst AND custom tools |
| **Procedure Parameters** | Prefix with `p_`, reference with colon (`:p_param`) |
| **Python Parsing** | Use `json.loads()` to extract full response details |

## Tool Type Reference

| Type | Purpose | Example |
|------|---------|---------|
| `cortex_analyst_text_to_sql` | SQL on structured data | Analyst |
| `cortex_search` | Search unstructured docs | Search |
| `generic` | Custom procedures | InventoryLookup, PriceCalculator |
| `web_search` | External WWW search | WebSearch |
| `data_to_chart` | Auto-generate visualizations | DataToChart |
