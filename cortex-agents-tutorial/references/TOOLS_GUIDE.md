# Cortex Agent Tools Guide

Detailed reference for configuring each tool type in Cortex Agents.

**Updated: February 2026** - Includes best practices from Snowflake's agent building guide.

---

## Tool Types Overview

| Tool Type | Purpose | Example Name |
|-----------|---------|--------------|
| `cortex_analyst_text_to_sql` | Query structured data | SalesDataAnalytics |
| `cortex_search` | Search unstructured data | ProductDocSearch |
| `generic` | Custom business logic | InventoryLookup, PriceCalculator |
| `web_search` | External WWW search | WebSearch |
| `data_to_chart` | Generate visualizations | DataToChart |

---

## Best Practice: Tool Naming

**Use Domain + Function pattern:**

| Good | Bad |
|------|-----|
| `SalesDataAnalytics` | `DataTool` |
| `ProductDocSearch` | `Search` |
| `InventoryLookup` | `Tool1` |
| `PriceCalculator` | `CustomTool` |

---

## Best Practice: Tool Descriptions

Every tool description should include:

1. **What it does** - Clear purpose
2. **Data coverage** - What data it accesses
3. **When to Use** - Specific scenarios
4. **When NOT to Use** - Redirect to other tools

### Example: Good Description

```yaml
description: |
  Analyzes structured sales data including revenue, quantities, products, regions, and customer segments.
  
  Data Coverage: Sales transactions with product details, regional breakdown, and customer information.
  
  When to Use:
  - Questions about sales metrics (revenue, quantity, averages)
  - Regional or product comparisons
  - Trend analysis over time periods
  
  When NOT to Use:
  - Product troubleshooting (use ProductDocSearch)
  - Real-time inventory levels (use InventoryLookup)
  - External market data (use WebSearch)
```

### Example: Bad Description

```yaml
description: "Get sales data"
```

---

## Cortex Analyst Tool

### Configuration (New Format)

```yaml
tools:
  - tool_spec:
      type: "cortex_analyst_text_to_sql"
      name: "SalesDataAnalytics"
      description: |
        Analyzes sales data including revenue, products, regions, and customer segments.
        
        When to Use: Questions about sales metrics and revenue.
        When NOT to Use: Product documentation questions.
    tool_resources:
      semantic_view:
        - database: "CORTEX_AGENTS_LAB"
          schema: "TUTORIAL"
          name: "SALES_SEMANTIC_VIEW"
```

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `semantic_view.database` | Yes | Database containing the view |
| `semantic_view.schema` | Yes | Schema containing the view |
| `semantic_view.name` | Yes | Semantic view name (UPPERCASE) |

---

## Cortex Search Tool

### Configuration

```yaml
tools:
  - tool_spec:
      type: "cortex_search"
      name: "ProductDocSearch"
      description: |
        Searches product documentation for features, specifications, and troubleshooting.
        
        When to Use: Product features, "how to" questions, troubleshooting.
        When NOT to Use: Sales metrics (use SalesDataAnalytics).
    tool_resources:
      cortex_search_service:
        - database: "CORTEX_AGENTS_LAB"
          schema: "TUTORIAL"
          service_name: "PRODUCT_SEARCH_SERVICE"
          max_results: 3
          retrieval_columns: ["content"]
          filter: {}
```

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `database` | Yes | Database containing the service |
| `schema` | Yes | Schema containing the service |
| `service_name` | Yes | Cortex Search service name |
| `max_results` | No | Maximum documents to return (default: 10) |
| `retrieval_columns` | No | Columns to retrieve |
| `filter` | No | Filter criteria |

---

## Custom Tools (Generic)

**IMPORTANT**: The Snowsight UI custom tools picker only lists **stored procedures**, not UDFs.
Always create procedures for custom agent tools.

### Procedure-Based Tool

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
    SELECT TO_VARCHAR(quantity_in_stock) || ' units in stock'
      INTO :result
    FROM CORTEX_AGENTS_LAB.TUTORIAL.INVENTORY
    WHERE UPPER(product_name) = UPPER(:p_product_name);
    RETURN result;
END;
$$;
```

**Key requirements**:
- Use `CREATE PROCEDURE`, not `CREATE FUNCTION`
- Use `EXECUTE AS CALLER` for proper privilege flow
- Prefix parameters with `p_` to avoid column name collisions
- Reference parameters with colon prefix (`:p_product_name`) inside SQL
- Use `DECLARE`/`BEGIN`/`END` block with `$$` delimiters

Then add via Snowsight UI:
- Resource type: **Procedure**
- Select: `CORTEX_AGENTS_LAB.TUTORIAL.CHECK_INVENTORY_PROC`
- Warehouse: `COMPUTE_WH`

### Multi-Argument Procedure

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
        ' | Total: $' || TO_VARCHAR(unit_price * :p_quantity, '999,999.99')
      INTO :result
    FROM CORTEX_AGENTS_LAB.TUTORIAL.SALES
    WHERE UPPER(product_name) = UPPER(:p_product_name)
    LIMIT 1;
    RETURN result;
END;
$$;
```

---

## Built-in: Web Search Tool

```yaml
tools:
  - tool_spec:
      type: "web_search"
      name: "WebSearch"
      description: |
        Searches the public internet for real-time information.
        
        When to Use:
        - Current market trends and pricing
        - Competitor information
        - Industry news and external data
        
        When NOT to Use:
        - Internal sales data (use SalesDataAnalytics)
        - Product documentation (use ProductDocSearch)
```

**Note**: Web search must be enabled at the account level in Snowsight (AI & ML → Agents → Settings).

---

## Built-in: Data to Chart Tool

```yaml
tools:
  - tool_spec:
      type: "data_to_chart"
      name: "DataToChart"
      description: |
        Automatically generates Vega-Lite visualizations from data.
        
        When to Use:
        - User requests charts, graphs, or visualizations
        - Comparing data visually
        
        When NOT to Use:
        - Simple numeric answers
        - Single data points
```

---

## Agent Instructions

### Best Practice: Separate Orchestration and Response

```yaml
agent_instructions:
  orchestration_instructions: |
    **Role:** You are "Sales Assistant", an AI-powered assistant.
    
    **Tool Selection Guidelines:**
    - For sales metrics: Use "SalesDataAnalytics"
    - For product docs: Use "ProductDocSearch"
    - For stock levels: Use "InventoryLookup"
    - For pricing: Use "PriceCalculator"
    - For external info: Use "WebSearch"
    - For visualizations: Use "DataToChart"
    
    **Boundaries:**
    - You do NOT access customer PII
    - You do NOT provide financial advice
    
  response_instructions: |
    **Style:**
    - Be concise and professional
    - Lead with the answer, then details
    
    **Data Presentation:**
    - Use tables for multi-row data
    - Include units with numbers
```

---

## Complete 7-Tool Agent Example

```sql
CREATE OR REPLACE AGENT sales_assistant
  COMMENT = 'AI assistant with 7 tools'
  FROM SPECIFICATION
  $$
  agent_instructions:
    orchestration_instructions: |
      **Role:** Sales intelligence assistant
      **Tool Selection:** Match question type to tool
      **Boundaries:** No PII, no financial advice
    response_instructions: |
      **Style:** Concise, data-driven
      **Format:** Tables for data, units always
  
  tools:
    - tool_spec:
        type: "cortex_analyst_text_to_sql"
        name: "SalesDataAnalytics"
        description: |
          Analyzes sales data. When to Use: Metrics, revenue. When NOT: Docs, inventory.
      tool_resources:
        semantic_view:
          - database: "DB"
            schema: "SCHEMA"
            name: "SALES_SEMANTIC_VIEW"

    - tool_spec:
        type: "cortex_search"
        name: "ProductDocSearch"
        description: |
          Searches docs. When to Use: Features, troubleshooting. When NOT: Sales data.
      tool_resources:
        cortex_search_service:
          - database: "DB"
            schema: "SCHEMA"
            service_name: "PRODUCT_SEARCH_SERVICE"
            max_results: 3
            retrieval_columns: ["content"]
            filter: {}

    - tool_spec:
        type: "generic"
        name: "InventoryLookup"
        description: "Check stock. When to Use: Availability. When NOT: Sales."
        function:
          name: "DB.SCHEMA.CHECK_INVENTORY_PROC"
          arguments:
            - name: "p_product_name"
              type: "string"

    - tool_spec:
        type: "generic"
        name: "PriceCalculator"
        description: "Calculate prices with discounts."
        function:
          name: "DB.SCHEMA.CALCULATE_PRICE_PROC"
          arguments:
            - name: "p_product_name"
              type: "string"
            - name: "p_quantity"
              type: "number"

    - tool_spec:
        type: "generic"
        name: "ProductSummary"
        description: "Quick product overview."
        function:
          name: "DB.SCHEMA.GET_PRODUCT_SUMMARY_PROC"
          arguments:
            - name: "p_product_name"
              type: "string"

    - tool_spec:
        type: "web_search"
        name: "WebSearch"
        description: "External WWW search for market info."

    - tool_spec:
        type: "data_to_chart"
        name: "DataToChart"
        description: "Auto-generate visualizations."
  $$;
```

---

## Quick Reference

| Tool Type | Key Configuration |
|-----------|------------------|
| `cortex_analyst_text_to_sql` | `tool_resources.semantic_view` with database, schema, name |
| `cortex_search` | `tool_resources.cortex_search_service` with service_name |
| `generic` | `function.name` and `function.arguments` |
| `web_search` | No tool_resources needed |
| `data_to_chart` | No tool_resources needed |
