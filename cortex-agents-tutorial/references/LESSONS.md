# Cortex Agents Tutorial - Lesson SQL Reference

This file contains all SQL code for the tutorial, organized by lesson. Execute each section step-by-step, explaining as you go.

**IMPORTANT**: This file contains TESTED, WORKING SQL syntax as of March 2026.
**CRITICAL**: `tool_resources` is NOT supported via CREATE AGENT SQL. Add tools via Snowsight UI.

---

## Lesson 1: Setup & Prerequisites

**Learning Objective**: Set up the environment and verify Cortex Agent access.

### Step 1.1: Check Current Context

```sql
SELECT CURRENT_ROLE(), CURRENT_WAREHOUSE(), CURRENT_DATABASE(), CURRENT_SCHEMA();
```

**Explain**: Let's see what role, warehouse, database, and schema we're currently using.

### Step 1.2: Set Up Environment

```sql
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;

CREATE DATABASE IF NOT EXISTS CORTEX_AGENTS_LAB;
USE DATABASE CORTEX_AGENTS_LAB;

CREATE SCHEMA IF NOT EXISTS TUTORIAL;
USE SCHEMA TUTORIAL;
```

### Step 1.3: Verify Cortex Access

```sql
SELECT SNOWFLAKE.CORTEX.COMPLETE('claude-3-5-sonnet', 'Say hello in one word');
```

**Explain**: This quick test confirms your account has Cortex AI enabled.

---

## Lesson 2: Create Sample Data

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

## Lesson 3: Create Semantic View for Cortex Analyst

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

### Step 3.2: Verify and Test

```sql
SHOW SEMANTIC VIEWS;

SELECT * FROM SEMANTIC_VIEW(
    CORTEX_AGENTS_LAB.TUTORIAL.SALES_SEMANTIC_VIEW
    METRICS total_sales, total_quantity
    DIMENSIONS region
)
ORDER BY total_sales DESC;
```

---

## Lesson 4: Create Cortex Search Service

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

---

## Lesson 5: Build Your First Agent

**Learning Objective**: Create a Cortex Agent with instructions only (tools added via UI).

### CRITICAL: tool_resources NOT supported in SQL - Add tools via Snowsight UI!

### Step 5.1: Create Minimal Agent

```sql
CREATE OR REPLACE AGENT sales_assistant
  COMMENT = 'AI assistant for sales analytics and product documentation'
  FROM SPECIFICATION
  $$
  instructions:
    orchestration: |
      You are Sales Assistant, an AI-powered sales intelligence assistant.
      For sales metrics, revenue, trends: Use SalesDataAnalytics
      For product documentation and troubleshooting: Use ProductDocSearch
    response: |
      Be concise and professional.
      Lead with the direct answer, then provide supporting details.
  $$;

SHOW AGENTS;
```

**Explain**: This creates the agent with instructions. We'll add tools via the Snowsight UI next.

---

## Lesson 6: Add Tools via Snowsight UI

**Learning Objective**: Configure agent tools through the UI (SQL doesn't support tool_resources).

### Step 6.1: Create Custom Tool Procedures First

**IMPORTANT**: Custom agent tools require **stored procedures**, not UDFs. The Snowsight UI
for adding custom tools only lists procedures. UDFs will not appear in the tool picker.

Also note: SQL UDF parameter names like `check_inventory.product_name` cause `invalid identifier`
errors. Procedures with `p_` prefixed parameters and `$$` delimited bodies avoid this.

```sql
-- Inventory Check
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

-- Price Calculator with discounts
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

-- Product Summary
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

### Step 6.2: Verify Procedures Work

Before going to the UI, confirm the procedures are callable:

```sql
CALL check_inventory_proc('Standing Desk');
CALL calculate_price_proc('Laptop Pro', 50);
CALL get_product_summary_proc('Laptop Pro');
```

### Step 6.3: Add Tools via Snowsight UI

Now that all procedures exist, go to the UI to wire them up.

**Navigate to**: AI & ML → Studio → SALES_ASSISTANT → Edit → Tools

**Add each tool**:

1. **Cortex Analyst** (SalesDataAnalytics)
   - Click "+ Add" next to Cortex Analyst
   - Name: `SalesDataAnalytics`
   - Select semantic view: `CORTEX_AGENTS_LAB.TUTORIAL.SALES_SEMANTIC_VIEW`
   - Warehouse: `COMPUTE_WH`
   - Description: "Analyzes sales data including revenue, products, regions"

2. **Cortex Search** (ProductDocSearch)
   - Click "+ Add" next to Cortex Search Services
   - Name: `ProductDocSearch`
   - Select service: `CORTEX_AGENTS_LAB.TUTORIAL.PRODUCT_SEARCH_SERVICE`
   - Description: "Searches product documentation"

3. **Custom Tools** (CHECK_INVENTORY_PROC, CALCULATE_PRICE_PROC, GET_PRODUCT_SUMMARY_PROC)
   - Click "+ Add" next to Custom tools
   - Resource type: **Procedure** (NOT Function -- UDFs will not appear here)
   - Select each procedure created above
   - Warehouse: `COMPUTE_WH`

4. **Save** the agent

---

## Lesson 7: Test All Tools

**Learning Objective**: Run queries and observe how the agent orchestrates across tools.

### Understanding DATA_AGENT_RUN

Before we start testing, let's clarify what `DATA_AGENT_RUN` is:

```
CREATE AGENT       = Builds the agent (brain + instructions)
Snowsight UI       = Gives it tools (hands)
DATA_AGENT_RUN     = Sends it a message (mouth)
```

**`DATA_AGENT_RUN` is NOT an agent itself** -- it's the SQL function that delivers your question to an agent and brings back the answer. The agent (`SALES_ASSISTANT`) is the object that has the instructions, tools, and orchestration logic.

Think of it like stored procedures:
- `CREATE PROCEDURE` defines the logic → `CREATE AGENT` defines the agent
- `CALL my_procedure()` executes it → `DATA_AGENT_RUN('my_agent', ...)` talks to it

Without the agent object existing first, `DATA_AGENT_RUN` has nothing to call. Without `DATA_AGENT_RUN`, the agent exists but has no way to receive questions via SQL.

**The call format is always the same** -- only the question text changes:

```sql
SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    '<database>.<schema>.<agent_name>',
    '{"messages": [{"role": "user", "content": [{"type": "text", "text": "<your question>"}]}]}'
);
```

You don't tell the agent which tool to use. The agent figures that out from:
1. Your question's intent
2. The orchestration instructions (Lesson 5)
3. The tool descriptions configured in Snowsight (Lesson 6)

Now let's test each tool.

### Test 1: Sales Analysis (SalesDataAnalytics)

```sql
SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'CORTEX_AGENTS_LAB.TUTORIAL.SALES_ASSISTANT',
    '{"messages": [{"role": "user", "content": [{"type": "text", "text": "What are the total sales by region?"}]}]}'
);
```

### Test 2: Documentation Search (ProductDocSearch)

```sql
SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'CORTEX_AGENTS_LAB.TUTORIAL.SALES_ASSISTANT',
    '{"messages": [{"role": "user", "content": [{"type": "text", "text": "How do I fix laptop overheating?"}]}]}'
);
```

### Test 3: Inventory Check (InventoryLookup)

```sql
SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'CORTEX_AGENTS_LAB.TUTORIAL.SALES_ASSISTANT',
    '{"messages": [{"role": "user", "content": [{"type": "text", "text": "How many Standing Desks do we have in stock?"}]}]}'
);
```

### Test 4: Price Calculation (PriceCalculator)

```sql
SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'CORTEX_AGENTS_LAB.TUTORIAL.SALES_ASSISTANT',
    '{"messages": [{"role": "user", "content": [{"type": "text", "text": "What would 50 Laptop Pro units cost with bulk discount?"}]}]}'
);
```

### Test 5: Product Summary (ProductSummary)

```sql
SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'CORTEX_AGENTS_LAB.TUTORIAL.SALES_ASSISTANT',
    '{"messages": [{"role": "user", "content": [{"type": "text", "text": "Give me a quick summary of Standing Desk performance"}]}]}'
);
```

### Test 6: Web Search (WebSearch)

```sql
SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'CORTEX_AGENTS_LAB.TUTORIAL.SALES_ASSISTANT',
    '{"messages": [{"role": "user", "content": [{"type": "text", "text": "What is the current market trend for standing desks?"}]}]}'
);
```

### Test 7: Data Visualization (DataToChart)

```sql
SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'CORTEX_AGENTS_LAB.TUTORIAL.SALES_ASSISTANT',
    '{"messages": [{"role": "user", "content": [{"type": "text", "text": "Create a bar chart showing revenue by region"}]}]}'
);
```

---

## Lesson 8: Verification & Cleanup

**Learning Objective**: Verify everything works and clean up tutorial objects.

### Step 8.1: Final Verification

```sql
SHOW AGENTS;
SHOW SEMANTIC VIEWS;
SHOW CORTEX SEARCH SERVICES;
SHOW TABLES;
SHOW PROCEDURES IN SCHEMA CORTEX_AGENTS_LAB.TUTORIAL;
```

### Step 8.2: Cleanup (Only if requested)

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
```

---

## Best Practices Summary

| Practice | Implementation |
|----------|---------------|
| **Descriptive Tool Names** | `SalesDataAnalytics` not `DataTool` |
| **When to Use / NOT Use** | Every tool description includes clear guidance |
| **Agent Instructions** | `orchestration_instructions` + `response_instructions` |
| **Tool Selection Guidelines** | Explicit routing in orchestration instructions |
| **Boundaries** | Clear limitations defined upfront |

## Tool Type Reference

| Type | Purpose | Example |
|------|---------|---------|
| `cortex_analyst_text_to_sql` | SQL on structured data | SalesDataAnalytics |
| `cortex_search` | Search unstructured docs | ProductDocSearch |
| `generic` | Custom procedures/functions | InventoryLookup, PriceCalculator |
| `web_search` | External WWW search | WebSearch |
| `data_to_chart` | Auto-generate visualizations | DataToChart |
