# RAG Patterns with Cortex Search

## What is RAG?

RAG (Retrieval Augmented Generation) is a technique that enhances LLM responses by first retrieving relevant data from a knowledge base, then passing that data as context to the LLM. This grounds the LLM's answer in your actual data instead of relying solely on its training knowledge.

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   User Query    │     │  Cortex Search   │     │  Cortex LLM     │
│                 │────>│  (Retrieval)     │────>│  (Generation)   │
│ "How do I get   │     │                 │     │                 │
│  a refund?"     │     │ Finds relevant   │     │ Generates answer│
│                 │     │ transcripts      │     │ using context   │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                              │                        │
                              ▼                        ▼
                        ┌───────────┐          ┌───────────────┐
                        │ Context:  │          │ "To get a     │
                        │ 3 related │          │ refund, call  │
                        │ support   │          │ support and   │
                        │ tickets   │          │ they will..." │
                        └───────────┘          └───────────────┘
```

### Why RAG?

| Without RAG | With RAG |
|-------------|----------|
| LLM answers from training data only | LLM answers from YOUR data |
| May hallucinate or give generic answers | Answers grounded in actual documents |
| No access to private/recent data | Uses your latest data |
| Can't cite sources | Can reference specific documents |

## Basic RAG Pattern

The fundamental pattern has three steps:

### Step 1: Retrieve

Search your knowledge base for relevant context.

```python
resp = search_service.search(
    query=user_question,
    columns=["transcript_text", "region"],
    limit=3
)
results = json.loads(resp.to_json())
```

### Step 2: Build Context

Combine the search results into a context string.

```python
context_parts = []
for r in results['results']:
    context_parts.append(r['transcript_text'])
context = "\n---\n".join(context_parts)
```

### Step 3: Generate

Pass the context and question to an LLM.

```python
prompt = f"""Based on the following information, answer the question.
If the answer is not found in the provided information, say so.

Information:
{context}

Question: {user_question}

Answer:"""

response = session.sql(
    "SELECT SNOWFLAKE.CORTEX.COMPLETE('mistral-large2', ?) AS response",
    params=[prompt]
).collect()
```

## SQL RAG Pattern

The entire RAG pipeline can be expressed in a single SQL query:

```sql
WITH search_results AS (
    SELECT PARSE_JSON(
        SNOWFLAKE.CORTEX.SEARCH_PREVIEW(
            'my_search_service',
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
        'Based on these support transcripts, answer the question.\n\n',
        'Transcripts:\n', context_text, '\n\n',
        'Question: How do I reset my password?\nAnswer:'
    )
) AS ai_response
FROM context;
```

### How It Works

1. **search_results CTE**: Calls SEARCH_PREVIEW to find the top 3 relevant transcripts
2. **context CTE**: Uses LATERAL FLATTEN to extract transcript texts from the JSON array, then concatenates them
3. **Final SELECT**: Passes the context to CORTEX.COMPLETE with a prompt template

## Python RAG Pattern

A reusable Python function:

```python
import json
from snowflake.core import Root
from snowflake.snowpark.context import get_active_session

session = get_active_session()
root = Root(session)

def rag_query(question, service_name, search_column, limit=3, filter=None):
    """Generic RAG function: search + generate."""
    
    # Retrieve
    service = (
        root.databases[session.get_current_database()]
        .schemas[session.get_current_schema()]
        .cortex_search_services[service_name]
    )
    
    search_params = {
        "query": question,
        "columns": [search_column],
        "limit": limit
    }
    if filter:
        search_params["filter"] = filter
    
    resp = service.search(**search_params)
    results = json.loads(resp.to_json())
    
    # Build context
    context = "\n---\n".join(
        r[search_column] for r in results['results']
    )
    
    # Generate
    prompt = f"""You are a helpful assistant. Answer the question based only on 
the provided context. If the answer is not in the context, say "I don't have 
enough information to answer that."

Context:
{context}

Question: {question}

Answer:"""
    
    response = session.sql(
        "SELECT SNOWFLAKE.CORTEX.COMPLETE('mistral-large2', ?) AS response",
        params=[prompt]
    ).collect()
    
    return response[0]['RESPONSE']
```

## Prompt Engineering for RAG

The prompt template is critical for good RAG results. Here are patterns that work well.

### System Instructions

Always tell the LLM its role and constraints:

```
You are a helpful customer support assistant for Tasty Bytes.
Answer questions based ONLY on the provided support transcripts.
If the answer is not found in the transcripts, say so clearly.
Do not make up information.
```

### Context Formatting

Structure the context so the LLM can parse it:

```
Past support conversations:
[North America - Billing] Customer was overcharged $15.99...
---
[Europe - Account] Customer unable to reset password...
---
[North America - Billing] Customer wants to cancel subscription...
```

Including metadata (region, category) helps the LLM give more specific answers.

### Question Placement

Place the question AFTER the context so the LLM processes context first:

```
Context:
{context}

Question: {question}

Answer:
```

### Instruction to Stay Grounded

Explicitly tell the LLM to only use provided context:

```
Answer the question based ONLY on the information provided above.
If the answer cannot be determined from the provided information,
respond with "I don't have enough information to answer that question."
```

## Handling "No Relevant Results"

When the search returns no good matches, the RAG system should handle it gracefully.

### Check Result Count

```python
results = json.loads(resp.to_json())

if not results.get('results'):
    return "I couldn't find any relevant information for your question."
```

### Let the LLM Decide

Include a fallback instruction in the prompt:

```
If none of the provided transcripts are relevant to the question,
respond with: "I don't have information about that topic in our 
support history. Please contact a live agent for assistance."
```

### Confidence Thresholds

For multi-index services with scoring, you can filter by relevance:

```python
# Only use results above a relevance threshold
relevant = [r for r in results['results'] if r.get('score', 1) > 0.5]
if not relevant:
    return "No sufficiently relevant results found."
```

## Multi-Turn Conversation

For chat-like experiences, maintain conversation history:

```python
conversation_history = []

def chat(question):
    # Add user message to history
    conversation_history.append(f"User: {question}")
    
    # Search with the latest question
    resp = service.search(
        query=question,
        columns=["transcript_text"],
        limit=3
    )
    results = json.loads(resp.to_json())
    context = "\n---\n".join(r['transcript_text'] for r in results['results'])
    
    # Include conversation history in prompt
    history_text = "\n".join(conversation_history[-6:])  # Last 3 exchanges
    
    prompt = f"""You are a support assistant. Use the context to answer.

Context:
{context}

Conversation so far:
{history_text}

Answer the user's latest question:"""
    
    response = session.sql(
        "SELECT SNOWFLAKE.CORTEX.COMPLETE('mistral-large2', ?) AS response",
        params=[prompt]
    ).collect()
    
    answer = response[0]['RESPONSE']
    conversation_history.append(f"Assistant: {answer}")
    return answer
```

## Choosing the Right LLM Model

Cortex COMPLETE supports multiple models. Choose based on your needs:

| Model | Speed | Quality | Best For |
|-------|-------|---------|----------|
| `mistral-large2` | Moderate | High | General RAG, complex questions |
| `llama3.1-70b` | Moderate | High | Detailed answers, reasoning |
| `llama3.1-8b` | Fast | Good | Simple Q&A, high throughput |
| `mistral-7b` | Fast | Good | Quick answers, cost-sensitive |

### Recommendations

- **For production RAG chatbots**: `mistral-large2` or `llama3.1-70b`
- **For high-volume, simple Q&A**: `llama3.1-8b`
- **For cost-sensitive prototyping**: `mistral-7b`

## SQL vs Python for RAG

### SQL RAG

**Strengths:**
- Self-contained in a single query
- Easy to embed in views, stored procedures, tasks
- No Python environment needed
- Good for scheduled or batch RAG

**Limitations:**
- Harder to manage conversation state
- Less flexible prompt construction
- SEARCH_PREVIEW is for testing, not production serving

### Python RAG

**Strengths:**
- Full programmatic control
- Easy to manage conversation history
- Dynamic prompt construction
- Production-ready Python API for search
- Integration with Streamlit, web apps

**Limitations:**
- Requires Python environment
- More code to write

### Recommendation

- **Prototyping and testing**: SQL (quick to iterate)
- **Production applications**: Python (more control, production search API)
- **Streamlit apps**: Python (natural fit)

## When to Use RAG vs Alternatives

| Approach | Use When |
|----------|----------|
| **RAG with Cortex Search** | You have text data in Snowflake and want grounded Q&A |
| **Cortex Agents** | You need multi-tool orchestration (search + SQL + functions) |
| **COMPLETE with inline context** | Context is small and static (no search needed) |
| **Fine-tuning** | You need the LLM to learn a specific style or domain deeply |
| **CLASSIFY_TEXT** | You need categorization, not free-form answers |

## Best Practices

### 1. Limit Context Size

More context isn't always better. 3-5 relevant results usually outperform 10+ results with noise.

```python
# Good: Focused context
limit=3

# Often worse: Too much context dilutes relevance
limit=20
```

### 2. Include Metadata in Context

Adding metadata helps the LLM give more precise answers:

```python
context_parts = []
for r in results['results']:
    context_parts.append(
        f"[{r['region']} | {r['product_category']} | {r['created_date']}]\n{r['transcript_text']}"
    )
```

### 3. Use Filters to Narrow Search

Pre-filter results to improve relevance:

```python
# If the user mentions a region, filter to that region
resp = service.search(
    query=question,
    columns=["transcript_text"],
    filter={"@eq": {"region": user_region}},
    limit=3
)
```

### 4. Instruct the LLM to Cite Sources

Ask the LLM to reference which transcripts it used:

```
When answering, indicate which transcript(s) you based your answer on.
```

### 5. Handle Edge Cases

- Empty search results: Return a helpful fallback message
- Ambiguous questions: Ask for clarification
- Off-topic questions: Let the LLM say it can't help with that
- Very long transcripts: Consider truncating before passing as context
