# Cortex Agent Tools Guide

Detailed reference for configuring each tool type in Cortex Agents.

**Updated: March 2026** - Reflects working `FROM SPECIFICATION` YAML format with top-level `tool_resources`.

---

## Tool Types Overview

| Tool Type | Purpose | Example Name |
|-----------|---------|--------------|
| `cortex_analyst_text_to_sql` | Query structured data | Analyst |
| `cortex_search` | Search unstructured data | Search |
| `generic` | Custom business logic | InventoryLookup |
| `web_search` | External WWW search | WebSearch |
| `data_to_chart` | Generate visualizations | DataToChart |

---

## Best Practice: Tool Naming

**Use clear, descriptive names:**

| Good | Bad |
|------|-----|
| `Analyst` or `SalesDataAnalytics` | `DataTool` |
| `Search` or `ProductDocSearch` | `Tool1` |
| `InventoryLookup` | `CustomTool` |
| `PriceCalculator` | `proc1` |

---

## Best Practice: Tool Descriptions

Every tool description should include:

1. **What it does** - Clear purpose
2. **Data coverage** - What data it accesses
3. **When to Use** - Specific scenarios

### Example: Good Description

```yaml
description: "Queries structured sales data by converting natural language to SQL. Covers revenue, quantities, products, regions, and customer segments."
```

### Example: Bad Description

```yaml
description: "Get sales data"
```

---

## Agent YAML Spec Structure

All tools are configured inside `CREATE AGENT ... FROM SPECIFICATION $$yaml$$`.
The YAML has these top-level keys:

```yaml
models:
  orchestration: claude-4-sonnet       # LLM for orchestration
orchestration:
  budget:
    seconds: 30                        # Max execution time
    tokens: 16000                      # Max token budget
instructions:
  system: "Overall behavior"           # System-level instructions
  orchestration: "Tool selection"      # How to pick tools
  response: "Output format"            # How to format answers
tools:                                 # Tool definitions (list)
  - tool_spec:
      type: "..."
      name: "..."
      description: "..."
tool_resources:                        # Tool configuration (map)
  ToolName:
    ...config for that tool...
```

**IMPORTANT**: `tool_resources` is a TOP-LEVEL key, NOT nested under `tool_spec`. Each entry is keyed by the tool's `name` and must match exactly.

---

## Cortex Analyst Tool

### Configuration

```yaml
tools:
  - tool_spec:
      type: "cortex_analyst_text_to_sql"
      name: "Analyst"
      description: "Queries structured sales data by converting natural language to SQL"

tool_resources:
  Analyst:
    semantic_view: "CORTEX_AGENTS_LAB.TUTORIAL.SALES_SEMANTIC_VIEW"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
```

### CRITICAL: execution_environment is Required

The `execution_environment` block tells the agent which warehouse to use for running generated SQL. Without it, `DATA_AGENT_RUN` fails with error 399504.

**WRONG** (silently fails):
```yaml
tool_resources:
  Analyst:
    semantic_view: "DB.SCHEMA.VIEW"
    warehouse: "COMPUTE_WH"          # Bare key -- does NOT work!
```

**CORRECT**:
```yaml
tool_resources:
  Analyst:
    semantic_view: "DB.SCHEMA.VIEW"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
```

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `semantic_view` | Yes | Fully qualified name: `"DB.SCHEMA.VIEW_NAME"` |
| `execution_environment.type` | Yes | Must be `"warehouse"` |
| `execution_environment.warehouse` | Yes | Warehouse name for SQL execution |

---

## Cortex Search Tool

### Configuration

```yaml
tools:
  - tool_spec:
      type: "cortex_search"
      name: "Search"
      description: "Searches product documentation and troubleshooting guides"

tool_resources:
  Search:
    name: "CORTEX_AGENTS_LAB.TUTORIAL.PRODUCT_SEARCH_SERVICE"
    max_results: "3"
```

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `name` | Yes | Fully qualified search service name: `"DB.SCHEMA.SERVICE_NAME"` |
| `max_results` | No | Maximum documents to return (as string) |

---

## Custom Tools (Generic)

**IMPORTANT**: Custom agent tools require **stored procedures**, not UDFs. The Snowsight UI custom tools picker only lists procedures. For inline YAML specs, use the `function` key.

### Inline YAML Configuration

```yaml
tools:
  - tool_spec:
      type: "generic"
      name: "InventoryLookup"
      description: "Check product stock levels"
      input_schema:
        type: "object"
        properties:
          p_product_name:
            type: "string"
            description: "The product name to check"
        required: ["p_product_name"]

tool_resources:
  InventoryLookup:
    type: "procedure"
    identifier: "CORTEX_AGENTS_LAB.TUTORIAL.CHECK_INVENTORY_PROC"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
```

### Creating the Procedure

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

---

## Built-in: Web Search Tool

```yaml
tools:
  - tool_spec:
      type: "web_search"
      name: "WebSearch"
      description: "Searches the public internet for real-time information, market trends, and external data"
```

**Note**: Web search must be enabled at the account level in Snowsight (AI & ML > Agents > Settings). No `tool_resources` needed.

---

## Built-in: Data to Chart Tool

```yaml
tools:
  - tool_spec:
      type: "data_to_chart"
      name: "DataToChart"
      description: "Automatically generates Vega-Lite visualizations from data"
```

No `tool_resources` needed.

---

## Agent Instructions

### Best Practice: Use system, orchestration, and response

```yaml
instructions:
  system: |
    You are a helpful assistant for a retail business.
    You can answer questions about sales data and product documentation.
  orchestration: |
    Use Analyst for any question about sales, revenue, quantities, or metrics.
    Use Search for product documentation, troubleshooting, or how-to questions.
  response: |
    Be concise and include relevant numbers or details from the tools.
    Format currency with dollar signs and commas.
```

**Key names**:
- `system` - Overall agent behavior and persona
- `orchestration` - Tool selection guidance
- `response` - Output formatting rules

---

## Complete 2-Tool Agent Example (Tutorial Pattern)

This is the working pattern from the tutorial notebook:

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

---

## Quick Reference

| Tool Type | Key Configuration |
|-----------|------------------|
| `cortex_analyst_text_to_sql` | `semantic_view: "DB.SCHEMA.VIEW"` + `execution_environment` |
| `cortex_search` | `name: "DB.SCHEMA.SERVICE"` + `max_results` |
| `generic` | `type: procedure`, `identifier`, `execution_environment` |
| `web_search` | No tool_resources needed |
| `data_to_chart` | No tool_resources needed |
