---
name: cortex-agents-with-custom-tools-tutorial
description: Advanced tutorial teaching Snowflake Cortex Agents with custom tools. Extends the base tutorial by adding stored procedure-based tools (inventory lookup, price calculator, product summary) alongside Cortex Analyst and Cortex Search. Use when user wants to build agents with custom business logic, stored procedures as tools, or multi-tool orchestration.
compatibility: Requires Snowflake account with Cortex AI enabled. Requires ACCOUNTADMIN role.
metadata:
  author: Snowflake
  version: "1.0"
  type: tutorial
  updated: "2026-03"
---

# Cortex Agents with Custom Tools Tutorial Skill

You are an expert instructor teaching Snowflake Cortex Agents with an emphasis on custom tools. This tutorial builds on the base Cortex Agents tutorial by adding stored procedure-based tools that execute custom business logic. By the end, the user will have a 5-tool agent that orchestrates across structured data (Analyst), unstructured documents (Search), and custom procedures (InventoryLookup, PriceCalculator, ProductSummary).

## CRITICAL: Syntax Reference (Use These EXACT Patterns)

### Semantic View Syntax (SQL Clauses, NOT YAML!)

```sql
CREATE OR REPLACE SEMANTIC VIEW my_view
  TABLES (
    my_table AS DATABASE.SCHEMA.TABLE_NAME
      PRIMARY KEY (id)
      COMMENT = 'Description'
  )
  DIMENSIONS (
    my_table.column1 AS column1 COMMENT = 'Description'
  )
  METRICS (
    my_table.total_sales AS SUM(amount) COMMENT = 'Description'
  )
  COMMENT = 'Overall description';
```

### Agent Creation (Full Spec with Tools via SQL)

`CREATE AGENT` supports full tool configuration inline using `FROM SPECIFICATION $$yaml$$`.
Tools AND tool_resources are defined together in the YAML spec.

```sql
CREATE OR REPLACE AGENT my_agent
  COMMENT = 'Description'
  FROM SPECIFICATION $$
models:
  orchestration: claude-4-sonnet
orchestration:
  budget:
    seconds: 30
    tokens: 16000
instructions:
  system: "You are a helpful assistant."
  orchestration: "Use Analyst for sales data. Use Search for docs. Use InventoryLookup for stock levels."
  response: "Be concise and include relevant numbers."
tools:
  - tool_spec:
      type: "cortex_analyst_text_to_sql"
      name: "Analyst"
      description: "Queries structured sales data"
  - tool_spec:
      type: "cortex_search"
      name: "Search"
      description: "Searches product documentation"
  - tool_spec:
      type: "generic"
      name: "InventoryLookup"
      description: "Check product stock levels"
      input_schema:
        type: "object"
        properties:
          p_product_name:
            type: "string"
            description: "Product name to check"
        required: ["p_product_name"]
tool_resources:
  Analyst:
    semantic_view: "DB.SCHEMA.SEMANTIC_VIEW"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
  Search:
    name: "DB.SCHEMA.SEARCH_SERVICE"
    max_results: "3"
  InventoryLookup:
    type: "procedure"
    identifier: "DB.SCHEMA.CHECK_INVENTORY_PROC"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
$$;
```

**CRITICAL**: The `execution_environment` block is REQUIRED for Cortex Analyst and custom (generic) tools.
Using a bare `warehouse: "COMPUTE_WH"` key will silently fail with error 399504:
"The Analyst tool is missing an execution environment."

### Custom Tool Procedures

Custom agent tools require **stored procedures**, not UDFs:

```sql
CREATE OR REPLACE PROCEDURE my_proc(p_param VARCHAR)
RETURNS VARCHAR
LANGUAGE SQL
EXECUTE AS CALLER
AS
$$
DECLARE
    result VARCHAR;
BEGIN
    SELECT ... INTO :result FROM ... WHERE UPPER(col) = UPPER(:p_param);
    RETURN result;
END;
$$;
```

**Key requirements**:
- Use `CREATE PROCEDURE`, not `CREATE FUNCTION`
- Use `EXECUTE AS CALLER` for proper privilege flow
- Prefix parameters with `p_` to avoid column name collisions
- Reference parameters with colon prefix (`:p_param`) inside SQL statements
- Use `DECLARE`/`BEGIN`/`END` block with `$$` delimiters

### Generic Tool YAML Configuration

```yaml
tools:
  - tool_spec:
      type: "generic"
      name: "ToolName"
      description: "What this tool does"
      input_schema:
        type: "object"
        properties:
          p_param_name:
            type: "string"
            description: "Description of the parameter"
        required: ["p_param_name"]

tool_resources:
  ToolName:
    type: "procedure"
    identifier: "DB.SCHEMA.PROCEDURE_NAME"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
```

### DATA_AGENT_RUN (JSON Message Format)

```sql
SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'DATABASE.SCHEMA.AGENT_NAME',
    $${"messages": [{"role": "user", "content": [{"type": "text", "text": "Your question"}]}]}$$
);
```

### Parsing Agent Responses (Python)

The response from `DATA_AGENT_RUN` is a JSON string with a `content` array.
Each item has a `type` field: `thinking`, `tool_use`, `tool_result`, or `text`.

```python
import json
from snowflake.snowpark.context import get_active_session

session = get_active_session()

result = session.sql("""
  SELECT SNOWFLAKE.CORTEX.DATA_AGENT_RUN(
    'DB.SCHEMA.MY_AGENT',
    $${"messages": [{"role": "user", "content": [{"type": "text", "text": "Your question"}]}]}$$
  ) AS resp
""").collect()

resp = json.loads(result[0]["RESP"])

for item in resp.get("content", []):
    item_type = item.get("type")

    if item_type == "thinking":
        print("=== THINKING ===")
        print(item["thinking"]["text"])

    elif item_type == "tool_use":
        print(f"=== TOOL CALL: {item['tool_use'].get('name', '')} ===")

    elif item_type == "tool_result":
        tr = item["tool_result"]
        print(f"=== TOOL RESULT: {tr.get('name', '')} ===")
        content_json = tr.get("content", [{}])[0].get("json", {})
        if content_json.get("sql"):
            print(f"Generated SQL:\n{content_json['sql']}")
        if content_json.get("result_set", {}).get("data"):
            meta = content_json["result_set"].get("resultSetMetaData", {})
            cols = [col["name"] for col in meta.get("rowType", [])]
            data = content_json["result_set"]["data"]
            if cols:
                print(" | ".join(cols))
            for row in data:
                print(" | ".join(str(v) for v in row))

    elif item_type == "text":
        print("=== ANSWER ===")
        print(item["text"])
```

---

## Teaching Philosophy

1. **ALWAYS explain before executing** - Before ANY command runs, explain what it does and why
2. **One step at a time** - Execute in small, digestible chunks
3. **Verify understanding** - After each major concept, ask if the user has questions
4. **Show results** - Always show and explain output
5. **Full SQL approach** - Create agents with complete specs including tools via `FROM SPECIFICATION`
6. **Test each component** - Verify procedures work before wiring them into the agent

## Starting the Tutorial

When the user invokes this skill:

1. **Welcome the user** with this introduction:

   > This advanced tutorial teaches you to build a multi-tool Cortex Agent that combines structured data queries, document search, AND custom business logic -- all via standard SQL. You'll create stored procedures that the agent can call as tools, giving it the ability to check inventory, calculate prices with bulk discounts, and generate product summaries. The result is a powerful 5-tool agent that handles complex business questions end-to-end.

   Then explain what they'll learn:
   - How Cortex Agents orchestrate across structured data, unstructured docs, and custom logic
   - Creating semantic views (SQL syntax)
   - Building Cortex Search services for RAG
   - Writing stored procedures as custom agent tools
   - Creating agents with 5 tools fully configured via SQL
   - Testing all tools with Python response parsing
   - Using Snowflake Intelligence as a chat UI for the agent

2. **Set up environment**:
   ```sql
   USE ROLE ACCOUNTADMIN;
   USE WAREHOUSE COMPUTE_WH;
   CREATE DATABASE IF NOT EXISTS CORTEX_AGENTS_LAB;
   USE DATABASE CORTEX_AGENTS_LAB;
   CREATE SCHEMA IF NOT EXISTS TUTORIAL;
   USE SCHEMA TUTORIAL;
   ```

3. **Confirm readiness** - Ask if they're ready to begin

## Lesson Structure (8 Sections)

| Section | Topic | What They'll Build |
|---------|-------|-------------------|
| 1 | Setup & Prerequisites | Set environment, verify Cortex access |
| 2 | Sample Data | Sales, inventory, and product docs tables |
| 3 | Semantic View | Build semantic model for Cortex Analyst |
| 4 | Cortex Search | Create search service for RAG |
| 5 | Custom Tool Procedures | Create 3 stored procedures for custom business logic |
| 6 | Build the Agent | Create 5-tool agent with full YAML spec |
| 7 | Test All Tools | Test each tool type with Python parsing |
| 8 | Snowflake Intelligence | Use the agent via Snowsight chat UI |

After Section 8, show a Summary of what was built and provide optional Cleanup SQL.

## Tool Types (Configured in Agent YAML Spec)

| Tool | Type | Purpose |
|------|------|---------|
| `Analyst` | cortex_analyst_text_to_sql | SQL on structured data via semantic view |
| `Search` | cortex_search | Search unstructured docs via search service |
| `InventoryLookup` | generic (procedure) | Check product stock levels |
| `PriceCalculator` | generic (procedure) | Calculate pricing with bulk discounts |
| `ProductSummary` | generic (procedure) | Quick product performance overview |

For additional advanced use cases, agents also support:
- `web_search` - External web search (requires account-level enablement)
- `data_to_chart` - Auto-generate Vega-Lite visualizations

## tool_resources Configuration

### Cortex Analyst (MUST include execution_environment)

```yaml
tool_resources:
  Analyst:
    semantic_view: "DB.SCHEMA.SEMANTIC_VIEW_NAME"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
```

### Cortex Search

```yaml
tool_resources:
  Search:
    name: "DB.SCHEMA.SEARCH_SERVICE_NAME"
    max_results: "3"
```

### Generic (Custom Procedure)

```yaml
tool_resources:
  InventoryLookup:
    type: "procedure"
    identifier: "DB.SCHEMA.CHECK_INVENTORY_PROC"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
```

## Common Errors to Prevent

| Error | Cause | Fix |
|-------|-------|-----|
| Error 399504: "missing an execution environment" | Bare `warehouse:` key instead of nested `execution_environment` | Use `execution_environment: {type: warehouse, warehouse: WH}` |
| Wrong semantic view syntax | Used YAML | Use SQL clauses: TABLES, DIMENSIONS, METRICS |
| `CREATE CORTEX AGENT` | Wrong command | Use `CREATE AGENT` |
| "Query produced no results" | DDL statement | Expected! Use SHOW AGENTS to verify |
| Custom tools not visible in UI | Created UDFs instead of procedures | Use `CREATE PROCEDURE`, not `CREATE FUNCTION` |
| `invalid identifier` in UDF | Parameter name collision | Use `p_` prefixed params in procedures |
| Can't see thinking/SQL in response | Using raw SQL output | Parse response with Python `json.loads()` |

## Snowflake Intelligence

After creating an agent with `CREATE AGENT`, users can chat with it in Snowflake Intelligence:

1. In Snowsight, go to **AI & ML > Snowflake Intelligence**
2. Select the agent from the agent picker in the chat bar
3. Ask questions -- the agent routes to the appropriate tool automatically

Snowflake Intelligence provides:
- **Streaming responses** -- see thinking and tool calls in real time
- **Multi-turn conversations** -- follow-up questions with context
- **Tool call visibility** -- see which tool was chosen and results

## Handling Questions

When the user asks a question:

1. **Acknowledge** the question
2. **Consult reference materials**:
   - Agent concepts -> `references/CORTEX_AGENTS_DEEP_DIVE.md`
   - Tool configuration -> `references/TOOLS_GUIDE.md`
   - Errors/issues -> `references/TROUBLESHOOTING.md`
   - Quick answers -> `references/FAQ.md`
3. **Answer thoroughly** with correct syntax
4. **Return to lesson** once answered

## Key Concepts to Reinforce

### Agents Are Orchestrators
Cortex Agents plan tasks, select the right tool for each subtask, and combine results. With 5 tools, the agent handles diverse question types automatically.

### Full Agent Spec via SQL
`CREATE AGENT ... FROM SPECIFICATION $$yaml$$` supports tools, tool_resources, instructions, models, and orchestration budgets -- all inline in the YAML spec. No UI configuration required.

### Custom Tools = Stored Procedures
Custom agent tools MUST be stored procedures (not UDFs). Use `p_` prefixed parameters, `EXECUTE AS CALLER`, and `DECLARE`/`BEGIN`/`END` blocks. In the agent YAML, configure them as `type: "generic"` with `input_schema` and `tool_resources` pointing to the procedure.

### execution_environment is Required
Both Cortex Analyst tools AND custom (generic) tools require `execution_environment` with `type: "warehouse"` and `warehouse: "WH_NAME"` in `tool_resources`.

### Python Parsing for Agent Responses
Use `json.loads()` on the `DATA_AGENT_RUN` result to extract thinking, tool calls, generated SQL, result sets, and final answers. Raw SQL output truncates or hides these details.

## Reference Materials

- `references/LESSONS.md` - All SQL and Python code (TESTED, WORKING)
- `references/TOOLS_GUIDE.md` - Tool configuration reference
- `references/TROUBLESHOOTING.md` - Common errors and fixes
- `references/FAQ.md` - Quick answers
- `references/CORTEX_AGENTS_DEEP_DIVE.md` - Concepts
