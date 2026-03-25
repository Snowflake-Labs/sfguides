# Cortex Agents Deep Dive

Comprehensive guide to understanding Cortex Agents architecture and concepts.

---

## How Agents Work

### The Orchestration Loop

```
User Query
    |
┌─────────────────────────────────────┐
│         PLANNING PHASE              │
│  - Parse user intent                │
│  - Identify required information    │
│  - Select appropriate tools         │
│  - Break into subtasks if needed    │
└─────────────────────────────────────┘
    |
┌─────────────────────────────────────┐
│         EXECUTION PHASE             │
│  - Call selected tool(s)            │
│  - Cortex Analyst -> SQL generation │
│  - Cortex Search -> document retrieval│
│  - Custom tools -> procedure calls  │
└─────────────────────────────────────┘
    |
┌─────────────────────────────────────┐
│         REFLECTION PHASE            │
│  - Evaluate results                 │
│  - Decide: enough info? need more?  │
│  - Loop back or proceed to response │
└─────────────────────────────────────┘
    |
┌─────────────────────────────────────┐
│         RESPONSE PHASE              │
│  - Synthesize findings              │
│  - Apply response instructions      │
│  - Format and deliver answer        │
└─────────────────────────────────────┘
```

### Example: Multi-Tool Query

**User**: "How did Laptop Pro sell last quarter and what are its key specs?"

**Agent thinking**:
1. This needs sales data (numbers) -> Use `Analyst` (Cortex Analyst)
2. This needs product specs (text) -> Use `Search` (Cortex Search)
3. Plan: Run both tools, then combine results

**Execution**:
1. Cortex Analyst generates SQL: `SELECT SUM(total_amount) FROM sales WHERE product_name = 'Laptop Pro' AND sale_date >= '2024-01-01'`
2. Cortex Search finds: Laptop Pro specifications document

**Response**: "Laptop Pro generated $71,499.45 in sales last quarter across 55 units. Key specs include a 15.6-inch 4K display, Intel i9 processor, 32GB RAM, and 12-hour battery life."

### Example: Custom Tool Query

**User**: "What's in stock for Laptop Pro and how much would 50 units cost?"

**Agent thinking**:
1. Stock levels -> Use `InventoryLookup` (custom procedure)
2. Pricing for 50 units -> Use `PriceCalculator` (custom procedure)
3. Plan: Run both tools, combine results

**Execution**:
1. InventoryLookup calls `CHECK_INVENTORY_PROC('Laptop Pro')` -> "Product: Laptop Pro, Stock: 150 units"
2. PriceCalculator calls `CALCULATE_PRICE_PROC('Laptop Pro', 50)` -> "Product: Laptop Pro, Qty: 50, Unit Price: $1299.99, Discount: 10%, Total: $58,499.55"

**Response**: "Laptop Pro has 150 units in stock. For an order of 50 units at $1,299.99 each, you'd get a 10% bulk discount, bringing the total to $58,499.55."

---

## Tool Types Explained

### Cortex Analyst (Text-to-SQL)

**Purpose**: Query structured data using natural language.

**How it works**:
1. Agent sends question to Cortex Analyst
2. Analyst reads semantic model (dimensions, measures, relationships)
3. Generates appropriate SQL query
4. Executes query against warehouse
5. Returns results to agent

**Best for**:
- Numerical analysis (sales, counts, aggregations)
- Trend analysis (time-based queries)
- Comparisons (A vs B, rankings)
- Filtering and grouping

**Configuration** (in `FROM SPECIFICATION` YAML):
```yaml
tools:
  - tool_spec:
      type: "cortex_analyst_text_to_sql"
      name: "Analyst"
      description: "Query sales data for revenue and trends"

tool_resources:
  Analyst:
    semantic_view: "DB.SCHEMA.SEMANTIC_VIEW"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
```

**CRITICAL**: The `execution_environment` block is required. A bare `warehouse:` key silently fails.

### Cortex Search (RAG)

**Purpose**: Retrieve relevant unstructured documents via semantic search.

**How it works**:
1. Agent sends query to Cortex Search
2. Query is embedded into vector
3. Vector similarity search against indexed documents
4. Top-k relevant chunks returned
5. Agent uses retrieved context for response

**Best for**:
- Documentation lookup
- FAQ/knowledge base queries
- Policy/procedure questions
- Any text-based information retrieval

**Configuration** (in `FROM SPECIFICATION` YAML):
```yaml
tools:
  - tool_spec:
      type: "cortex_search"
      name: "Search"
      description: "Search product documentation"

tool_resources:
  Search:
    name: "DB.SCHEMA.SEARCH_SERVICE"
    max_results: "3"
```

### Custom Tools (Generic)

**Purpose**: Execute arbitrary business logic via stored procedures.

**How it works**:
1. Agent determines custom tool is needed based on description and orchestration instructions
2. Extracts required parameters from user query using `input_schema`
3. Calls procedure with extracted parameters
4. Receives VARCHAR result
5. Incorporates into response

**Best for**:
- Complex business logic (pricing, discounts)
- Multi-table operations (inventory checks)
- Calculations not expressible in semantic models
- Pre-built business rules

**Configuration** (in `FROM SPECIFICATION` YAML):
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
    identifier: "DB.SCHEMA.CHECK_INVENTORY_PROC"
    execution_environment:
      type: "warehouse"
      warehouse: "COMPUTE_WH"
```

**Key requirements for stored procedures**:
- Use `CREATE PROCEDURE` (not `CREATE FUNCTION`)
- Use `EXECUTE AS CALLER`
- Prefix parameters with `p_` to avoid column name collisions
- Return `VARCHAR` with human-readable results
- `input_schema` parameter names must match procedure parameter names exactly

---

## Instructions Deep Dive

### Orchestration Instructions

Guide the agent's planning and tool selection.

**Good patterns**:
```
"Use Analyst for questions about revenue, quantities, or trends.
Use Search for specifications, troubleshooting, or documentation.
Use InventoryLookup for stock level checks.
Use PriceCalculator for pricing quotes with bulk discounts.
Use ProductSummary for quick product performance overviews.
If a question spans multiple tools, query each and combine the results."
```

**What to include**:
- When to use each tool
- How to handle ambiguous queries
- Priority order for overlapping capabilities
- Edge cases and exceptions

### Response Instructions

Control how the agent formats and delivers answers.

**Good patterns**:
```
"Be concise and data-driven. Format numbers with commas.
When citing documents, include the source.
If data is unavailable, say so clearly rather than guessing."
```

**What to include**:
- Tone and style (formal, casual, technical)
- Formatting preferences (bullets, tables, prose)
- Citation requirements
- Handling of uncertainty

---

## Threads and Conversation Context

### Why Threads Matter

Without threads:
- Each query is independent
- No memory of previous questions
- Follow-ups require full context

With threads:
- Agent remembers conversation history
- Follow-ups work naturally ("What about Europe?" after "Show me sales by region")
- Context accumulates across turns

### Thread Lifecycle

```
1. Create thread -> Get thread_id
2. Send message (parent_message_id = "0") -> Get message_id
3. Send follow-up (parent_message_id = previous message_id)
4. Continue conversation...
5. Thread expires after TTL
```

### Best Practices

- Create one thread per conversation
- Store thread_id in your application session
- Handle thread expiration gracefully (create new thread)
- Don't share threads between users

---

## Security Model

### Privilege Flow

```
User makes request
    |
Agent runs with CALLER's privileges
    |
Tool accesses data with caller's grants
    |
RBAC policies apply automatically
```

### Key Security Points

1. **Agent doesn't bypass permissions** - The caller must have access to underlying data
2. **Row access policies apply** - Users see only their permitted data
3. **Masking policies apply** - Sensitive data is masked per policy
4. **Custom tools run as caller** - Use `EXECUTE AS CALLER` for proper privilege flow

### Least Privilege Setup

```sql
-- Create dedicated role for agent users
CREATE ROLE agent_user_role;

-- Grant only necessary access
GRANT USAGE ON DATABASE mydb TO ROLE agent_user_role;
GRANT USAGE ON SCHEMA mydb.myschema TO ROLE agent_user_role;
GRANT USAGE ON AGENT mydb.myschema.my_agent TO ROLE agent_user_role;
GRANT SELECT ON SEMANTIC VIEW mydb.myschema.sales_view TO ROLE agent_user_role;
GRANT USAGE ON CORTEX SEARCH SERVICE mydb.myschema.docs_search TO ROLE agent_user_role;
GRANT USAGE ON PROCEDURE mydb.myschema.check_inventory_proc(VARCHAR) TO ROLE agent_user_role;
GRANT USAGE ON PROCEDURE mydb.myschema.calculate_price_proc(VARCHAR, NUMBER) TO ROLE agent_user_role;

-- Don't grant access to underlying tables directly!
```

---

## Performance Optimization

### Tool Selection Efficiency

- Write specific tool descriptions to avoid wrong tool selection
- Use orchestration instructions to guide common patterns
- Consider combining related data into fewer tools

### Query Performance

- Optimize semantic models for common query patterns
- Size warehouses appropriately for tool complexity
- Set reasonable timeouts to fail fast on bad queries

### Search Performance

- Index only necessary columns
- Use filters to narrow search scope
- Limit max_results to what's needed

---

## Common Architecture Patterns

### Pattern 1: Sales + Documentation Agent (2 tools)

```
┌─────────────────────────────────────┐
│            AGENT                    │
├─────────────────────────────────────┤
│  Tool 1: Cortex Analyst             │
│  └─ Semantic View (sales data)      │
│                                     │
│  Tool 2: Cortex Search              │
│  └─ Search Service (product docs)   │
└─────────────────────────────────────┘
```

### Pattern 2: Full Sales Assistant (5 tools)

```
┌─────────────────────────────────────┐
│         SALES ASSISTANT             │
├─────────────────────────────────────┤
│  Tool 1: Analyst                    │
│  └─ Semantic View (sales metrics)   │
│                                     │
│  Tool 2: Search                     │
│  └─ Search Service (product docs)   │
│                                     │
│  Tool 3: InventoryLookup            │
│  └─ Procedure (stock levels)        │
│                                     │
│  Tool 4: PriceCalculator            │
│  └─ Procedure (pricing + discounts) │
│                                     │
│  Tool 5: ProductSummary             │
│  └─ Procedure (performance stats)   │
└─────────────────────────────────────┘
```

### Pattern 3: Snowflake Intelligence Agent

```
Any agent created with CREATE AGENT can be used in Snowflake Intelligence:
    |
    ├─ Cortex Analyst -> Semantic view for structured queries
    ├─ Cortex Search -> Knowledge base for RAG
    └─ Custom tools -> Business-specific logic

Access via: AI & ML > Snowflake Intelligence > select agent
```

---

## Evolution: Building Better Agents

### Start Simple
1. One tool
2. Basic instructions
3. Test with representative queries

### Add Custom Tools
1. Identify logic that can't be expressed in semantic views
2. Create stored procedures with clear inputs/outputs
3. Add `generic` tools with descriptive `input_schema`
4. Update orchestration instructions for new tools

### Iterate Based on Usage
1. Monitor which tools are used
2. Identify gaps in tool coverage
3. Refine instructions for edge cases

### Scale Thoughtfully
1. Add tools for new use cases
2. Split large semantic models if needed
3. Create specialized agents for different domains
