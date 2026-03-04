# Cortex Search Deep Dive

## What is Cortex Search?

Cortex Search is a Snowflake-managed hybrid search engine that combines **vector (semantic) search** with **keyword search** to deliver high-quality, low-latency search over your text data. You create a search service, point it at your data, and Snowflake handles everything else: embedding generation, index building, index refreshes, and serving.

## How It Works Under the Hood

```
┌──────────────────────────────────────────────────────────┐
│                   Your Source Data                        │
│  "Customer called about internet dropping every few..."  ���
└──────────────────────────┬───────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────┐
│             CREATE CORTEX SEARCH SERVICE                 │
│                                                          │
│  1. Embedding Generation                                 │
│     Text → snowflake-arctic-embed-l-v2.0 → [0.12, ...]  │
│                                                          │
│  2. Keyword Index                                        │
│     Text → tokenize → inverted index                     │
│                                                          │
│  3. Hybrid Index                                         │
│     Combines vector + keyword for ranked results         │
└──────────────────────────┬───────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────┐
│                    Search Query                          │
│  "internet connection problems"                          │
│                                                          │
│  1. Query embedding generated                            │
│  2. Vector similarity search (semantic meaning)          │
│  3. Keyword matching (exact terms)                       │
│  4. Results merged and ranked                            │
│  5. Filters applied on ATTRIBUTES columns                │
└──────────────────────────┬───────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────┐
│                    JSON Response                         │
│  {                                                       │
│    "results": [                                          │
│      {                                                   │
│        "transcript_text": "Customer called about...",    │
│        "region": "North America"                         │
│      },                                                  │
│      ...                                                 │
│    ]                                                     │
│  }                                                       │
└──────────────────────────────────────────────────────────┘
```

## Hybrid Search: Why Both Vector and Keyword?

Cortex Search combines two complementary search approaches:

### Vector (Semantic) Search

- Converts text to high-dimensional vectors (embeddings)
- Finds results based on **meaning**, not exact words
- Query "internet problems" finds "Wi-Fi connectivity dropping" because they mean similar things
- Great for: natural language queries, fuzzy matching, concept search

### Keyword Search

- Traditional text matching using tokenized terms
- Finds results based on **exact terms** present in the text
- Query "AG2001" finds records containing that exact agent ID
- Great for: exact identifiers, specific terms, product names, codes

### Why Hybrid?

| Search Type | Finds "Wi-Fi issues" for query "internet problems"? | Finds "AG2001" for query "AG2001"? |
|-------------|------------------------------------------------------|-------------------------------------|
| Vector only | Yes (semantic match) | Maybe (depends on embedding) |
| Keyword only | No (different words) | Yes (exact match) |
| **Hybrid** | **Yes** | **Yes** |

Hybrid search gives you the best of both worlds.

## Embedding Models

When creating a Cortex Search Service, you can specify which embedding model to use for vector search.

### Available Models

| Model | Dimensions | Context Window (tokens) | Language Support | Description |
|-------|-----------|------------------------|-----------------|-------------|
| `snowflake-arctic-embed-m-v1.5` (default) | 768 | 512 | English-only | Most practical, English-only model. 110M parameters, fastest indexing times. |
| `snowflake-arctic-embed-l-v2.0` | 1024 | 512 | Multilingual | Price-performant multilingual model. 568M parameters, high quality on English and non-English datasets. |
| `snowflake-arctic-embed-l-v2.0-8k` | 1024 | 8192 | Multilingual | Same as above with an increased 8K token context window. |
| `voyage-multilingual-2` | 1024 | 32,000 | Multilingual | Voyage's multilingual model. High quality on both English and non-English datasets. |

### Default Behavior

- If you specify `EMBEDDING_MODEL`, that model is used
- If you omit it, Snowflake uses a default model (`snowflake-arctic-embed-m-v1.5`)
- The embedding model **cannot be changed** after service creation — you must recreate the service

### Choosing a Model

- **For English-only, fast indexing**: Use `snowflake-arctic-embed-m-v1.5` (default) — smallest model, lowest cost
- **For multilingual, high-quality search**: Use `snowflake-arctic-embed-l-v2.0` — best balance of quality and cost
- **For long documents (up to 8K tokens)**: Use `snowflake-arctic-embed-l-v2.0-8k` — same quality as above with a larger context window
- **For very long documents (up to 32K tokens)**: Use `voyage-multilingual-2` — largest context window, multilingual

## Single-Index vs Multi-Index Services

### Single-Index (Simple Syntax)

The most common approach. One column is searched with both keyword and vector search:

```sql
CREATE CORTEX SEARCH SERVICE my_service
  ON text_column
  ATTRIBUTES filter_col1, filter_col2
  WAREHOUSE = my_wh
  TARGET_LAG = '1 hour'
  EMBEDDING_MODEL = 'snowflake-arctic-embed-l-v2.0'
  AS (SELECT text_column, filter_col1, filter_col2 FROM my_table);
```

### Multi-Index (Advanced Syntax)

Separate control over which columns use text (keyword) vs vector search:

```sql
CREATE CORTEX SEARCH SERVICE my_service
  TEXT INDEXES name, address
  VECTOR INDEXES description (model = 'snowflake-arctic-embed-m-v1.5')
  WAREHOUSE = my_wh
  TARGET_LAG = '1 hour'
  AS (SELECT name, address, description FROM my_table);
```

**When to use multi-index:**
- You have multiple searchable columns with different search needs
- Some columns need only keyword search (e.g., names, addresses)
- Some columns need semantic search (e.g., descriptions, reviews)
- You want to bring your own vector embeddings

## TARGET_LAG and Refresh Behavior

TARGET_LAG controls how fresh the search index stays relative to the source data.

### How It Works

```
Source data updated at 10:00 AM
TARGET_LAG = '1 hour'
→ Index will refresh by ~11:00 AM
```

### Refresh Modes

| Mode | Behavior | When Used |
|------|----------|-----------|
| INCREMENTAL | Only processes changed rows | Default when change tracking is enabled |
| FULL | Rebuilds the entire index | When incremental isn't possible |

### Choosing TARGET_LAG

| Lag Setting | Use Case | Relative Cost |
|-------------|----------|---------------|
| `'1 minute'` | Real-time search, live dashboards | Highest |
| `'1 hour'` | Most business applications | Moderate |
| `'1 day'` | Batch-updated data, archives | Lowest |

### Requirements for Incremental Refresh

- Change tracking must be enabled on the base table
- The source query must be compatible with dynamic table incremental refresh
- If these aren't met, Snowflake falls back to full refresh

```sql
-- Enable change tracking
ALTER TABLE my_table SET CHANGE_TRACKING = TRUE;
```

## ATTRIBUTES vs Search Columns

### Search Column (ON clause)

The column that is indexed for searching. When you issue a query, Cortex Search looks for matches in this column.

```sql
ON transcript_text  -- This column is searched
```

### ATTRIBUTES

Columns that can be:
1. **Returned** in search results
2. **Filtered** on using filter operators (@eq, @contains, etc.)

```sql
ATTRIBUTES region, product_category  -- These can be filtered and returned
```

### Source Query Columns

All columns in the SELECT query are available to return in results. Only ATTRIBUTES columns can be filtered.

```
Source query columns: transcript_id, transcript_text, region, product_category, agent_id
Search column:        transcript_text (searchable)
ATTRIBUTES:           region, product_category (filterable + returnable)
Other columns:        transcript_id, agent_id (returnable only)
```

## Querying: Three APIs

### 1. SQL - SEARCH_PREVIEW

Best for: Testing, validation, SQL worksheets, notebooks.

```sql
SELECT PARSE_JSON(
    SNOWFLAKE.CORTEX.SEARCH_PREVIEW(
        'my_service',
        '{"query": "search text", "columns": ["col1"], "limit": 5}'
    )
)['results'] AS results;
```

**Important**: SEARCH_PREVIEW is for testing and validation. It operates only on string literals and has higher latency than the Python and REST APIs. Do not use it for production applications.

### 2. Python SDK

Best for: Applications, Streamlit apps, notebooks, automation.

```python
from snowflake.core import Root

root = Root(session)
service = root.databases["DB"].schemas["SCHEMA"].cortex_search_services["SVC"]
resp = service.search(query="search text", columns=["col1"], limit=5)
```

Requires `snowflake` package version 0.8.0 or later.

### 3. REST API

Best for: External applications, non-Python clients, microservices.

```bash
curl --location https://<ACCOUNT_URL>/api/v2/databases/<DB>/schemas/<SCHEMA>/cortex-search-services/<SERVICE>:query \
  --header 'Content-Type: application/json' \
  --header "Authorization: Bearer $PAT" \
  --data '{"query": "search text", "columns": ["col1"], "limit": 5}'
```

## Scoring and Relevance

Cortex Search returns results ranked by relevance. The ranking combines:

1. **Vector similarity**: How semantically close the document is to the query
2. **Keyword relevance**: How well the document matches the query terms
3. **Hybrid fusion**: A combined score from both signals

### Customizing Scoring

For multi-index services, you can customize scoring with `scoring_config`:

```python
resp = service.search(
    query="search text",
    columns=["col1"],
    scoring_config={
        "weights": {
            "texts": 0.3,
            "vectors": 0.5,
            "reranker": 0.2
        }
    },
    limit=5
)
```

## Comparison with Other Search Methods

| Method | Semantic? | Managed? | Setup | Best For |
|--------|-----------|----------|-------|----------|
| **Cortex Search** | Yes (hybrid) | Fully managed | 1 SQL statement | Production search, RAG |
| `LIKE` / `ILIKE` | No | N/A | None | Simple pattern matching |
| `CONTAINS` | No | N/A | None | Substring matching |
| Custom embeddings + vector search | Yes | Self-managed | Significant | Full control needed |
| Full-text search (external) | Keyword only | Varies | Moderate | Legacy systems |

### When to Use Cortex Search

- You want semantic (meaning-based) search
- You need a managed, low-maintenance solution
- You're building RAG applications
- You want hybrid search without managing embeddings
- Your data lives in Snowflake

### When to Consider Alternatives

- Simple exact-match lookups → Use `LIKE` or `WHERE col = 'value'`
- You need sub-10ms latency → Consider a dedicated search engine
- You need full control over embeddings and ranking → Build custom with `AI_EMBED` and vector operations
