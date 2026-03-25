# Cortex Agents FAQ

Frequently asked questions about Snowflake Cortex Agents.

---

## General Questions

### What is a Cortex Agent?

A Cortex Agent is an AI-powered orchestrator that plans tasks, selects appropriate tools (Cortex Analyst, Cortex Search, custom procedures), executes them, and synthesizes responses. Unlike simple LLM calls, agents can handle complex multi-step queries.

### How is an Agent different from calling COMPLETE() directly?

| Direct LLM Call | Cortex Agent |
|-----------------|--------------|
| Single prompt/response | Multi-step orchestration |
| No data access | Connects to your data via tools |
| Stateless | Maintains context via threads |
| Generic responses | Grounded in enterprise data |

### What models can I use for orchestration?

- `auto` (recommended) - Snowflake selects the best available model
- `claude-4-sonnet`
- `claude-sonnet-4-5`
- `claude-3-5-sonnet`
- `openai-gpt-5`
- `openai-gpt-4-1`

### Do I need cross-region inference?

If your preferred model isn't available in your region, yes. Check the docs for regional availability.

---

## Tool Questions

### What tools can an agent use?

1. **Cortex Analyst** (`cortex_analyst_text_to_sql`) - Query structured data via semantic views
2. **Cortex Search** (`cortex_search`) - Search unstructured data via search services
3. **Custom Tools** (`generic`) - Call stored procedures or UDFs
4. **System Execute SQL** (`system_execute_sql`) - Run dynamic SQL
5. **Web Search** - Search the web (requires account-level enablement)

### How does the agent choose which tool to use?

The agent uses the tool descriptions and orchestration instructions to decide. Good descriptions are critical:

```json
// Good - specific about when to use
"description": "Query sales data for revenue, quantities, and trends. Use for numerical analysis."

// Bad - too vague
"description": "Get data from the database"
```

### Can an agent use multiple tools in one query?

Yes! The agent can plan multi-step tasks that use different tools. For example:
- User asks: "How did Laptop Pro sell and what are its specs?"
- Agent: Uses Cortex Analyst for sales data, then Cortex Search for specs

### What's the difference between Cortex Analyst and Cortex Search?

| Cortex Analyst | Cortex Search |
|----------------|---------------|
| Structured data (tables) | Unstructured data (text, docs) |
| Generates SQL queries | Vector similarity search |
| Requires semantic view | Requires search service |
| Returns query results | Returns relevant documents |

---

## Configuration Questions

### What are the instruction types?

In the `FROM SPECIFICATION` YAML, use the `instructions` key with three sub-keys:

- **system**: Overall agent behavior and persona
- **orchestration**: Guides tool selection and planning
- **response**: Controls output tone, format, style

```yaml
instructions:
  system: "You are a helpful assistant."
  orchestration: "Use Analyst for sales data. Use Search for documentation."
  response: "Be concise and include relevant numbers."
```

### How do I make my agent more accurate?

1. Write detailed tool descriptions
2. Add specific orchestration instructions
3. Use well-designed semantic models
4. Test with representative questions
5. Iterate based on results

### Can I limit what data the agent can access?

Yes, through standard Snowflake RBAC:
- The agent runs with the caller's privileges
- Grant only necessary access to semantic views, search services, and procedures
- Use row access policies and masking policies

---

## Thread Questions

### What is a thread?

A thread maintains conversation context across multiple interactions. Without threads, each query is independent.

### How long do threads persist?

Threads have a TTL (time-to-live). Check the docs for current limits.

### Do I need to use threads?

Not for single-turn queries, but recommended for:
- Multi-turn conversations
- Follow-up questions
- Context-dependent queries

---

## Snowflake Intelligence Questions

### How do I use my agent in Snowflake Intelligence?

Agents created with `CREATE AGENT` in any database/schema can be used in Snowflake Intelligence:

1. In Snowsight, go to **AI & ML > Snowflake Intelligence**
2. Select your agent from the agent picker in the chat bar
3. Ask questions -- the agent routes to the appropriate tool automatically

### Why isn't my agent showing in Snowflake Intelligence?

1. Verify the agent exists: `SHOW AGENTS IN SCHEMA mydb.myschema;`
2. Check USAGE privilege is granted on the agent
3. Ensure user has a default role and warehouse set
4. The agent picker may take a moment to refresh after creation

### What does Snowflake Intelligence provide?

- **Streaming responses** -- see the agent's thinking and tool calls in real time
- **Multi-turn conversations** -- follow-up questions with context
- **Tool call visibility** -- see which tool was chosen and the results
- No extra code needed -- works with any `CREATE AGENT` object

---

## Cost Questions

### How is agent usage charged?

- **Orchestration**: Token-based (like COMPLETE())
- **Cortex Analyst**: Token-based per query
- **Cortex Search**: Based on index size and persistence
- **Custom tools**: Warehouse compute time

### How do I minimize costs?

1. Use appropriate model (don't always use the largest)
2. Optimize semantic views for common queries
3. Set reasonable query timeouts
4. Use smaller warehouses for simple tools

---

## Quick Reference

### Essential Commands

```sql
-- Create agent
CREATE AGENT mydb.myschema.agent_name FROM SPECIFICATION $$ ... $$;

-- List agents
SHOW AGENTS IN SCHEMA mydb.myschema;

-- Describe agent
DESC AGENT mydb.myschema.agent_name;

-- Update agent (use CREATE OR REPLACE)
CREATE OR REPLACE AGENT mydb.myschema.agent_name FROM SPECIFICATION $$ ... $$;

-- Drop agent
DROP AGENT mydb.myschema.agent_name;

-- Grant access
GRANT USAGE ON AGENT mydb.myschema.agent_name TO ROLE analyst;
```

### Minimal Agent Spec

```yaml
models:
  orchestration: auto
instructions:
  system: "You are a helpful assistant."
tools: []
```

---

## Getting the Latest Documentation

| Topic | URL |
|-------|-----|
| Cortex Agents Overview | https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents |
| Managing Agents | https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents-manage |
| Agent Tutorials | https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents-tutorials |
| REST API | https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents-rest-api |
| Snowflake Intelligence | https://docs.snowflake.com/user-guide/snowflake-cortex/snowflake-intelligence |
