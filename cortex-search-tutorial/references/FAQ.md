# Frequently Asked Questions

## Basic Questions

### What is Cortex Search?

Cortex Search is a Snowflake-managed hybrid search engine that combines vector (semantic) search with keyword search. You point it at your text data, and it builds and maintains a search index automatically. It powers search experiences and RAG (Retrieval Augmented Generation) applications.

### How is Cortex Search different from LIKE or CONTAINS?

| Feature | LIKE / ILIKE | CONTAINS | Cortex Search |
|---------|-------------|----------|---------------|
| Match type | Pattern matching | Substring | Semantic + keyword |
| Finds synonyms | No | No | Yes |
| Understands meaning | No | No | Yes |
| Needs exact words | Yes | Yes | No |
| Managed index | No | No | Yes |
| Latency | Varies (table scan) | Varies (table scan) | Low (indexed) |

Example: Query "internet problems" with LIKE won't find "Wi-Fi connectivity dropping". Cortex Search will, because it understands the semantic similarity.

### Do I need to manage embeddings?

No. Cortex Search handles embedding generation, storage, and indexing automatically. You choose an embedding model when creating the service, and Snowflake does the rest.

### Do I need to train a model?

No. Cortex Search uses pre-trained embedding models (like `snowflake-arctic-embed-l-v2.0`) out of the box. No training data, no fine-tuning, no model deployment.

### What is hybrid search?

Hybrid search combines two techniques:
- **Vector search**: Finds semantically similar results (understands meaning)
- **Keyword search**: Finds exact term matches

By combining both, you get results that match both the meaning and the specific terms in your query.

## Service Management

### How many services can I create?

There is no hard limit on the number of services per account. Each service consumes compute (for refreshes) and storage (for the index), so create services based on your actual search needs.

### Can I update the source query after creation?

Yes, use ALTER CORTEX SEARCH SERVICE:

```sql
ALTER CORTEX SEARCH SERVICE my_service SET
  AS (SELECT text_col, new_col FROM updated_table);
```

Note that changing the source query may trigger a full index rebuild.

### Can I change the embedding model after creation?

No. The embedding model is fixed at creation time. To change it, you must drop and recreate the service.

### What happens when source data changes?

If change tracking is enabled and TARGET_LAG is set, Cortex Search will automatically refresh the index to include new, updated, or deleted rows. The refresh happens within the TARGET_LAG window.

### Can I suspend a service to save costs?

Yes:

```sql
-- Suspend (stops refreshes, service still queryable with stale data)
ALTER CORTEX SEARCH SERVICE my_service SUSPEND;

-- Resume
ALTER CORTEX SEARCH SERVICE my_service RESUME;
```

### How do I drop a service?

```sql
DROP CORTEX SEARCH SERVICE my_service;
```

## Querying

### What is the maximum number of results I can get?

The `limit` parameter supports up to 1000 results per query. The default is 10.

### How does filtering work?

Filters narrow search results based on ATTRIBUTES columns. They support:
- `@eq`: Exact equality
- `@contains`: Array contains
- `@gte` / `@lte`: Range comparisons (numeric, date)
- `@and` / `@or` / `@not`: Logical operators

See `references/FILTER_SYNTAX.md` for detailed examples.

### Can I search multiple columns?

With single-index services, one column is the search target (the `ON` column). With multi-index services, you can define multiple searchable columns using `TEXT INDEXES` and `VECTOR INDEXES`.

### What's the difference between SEARCH_PREVIEW and the Python API?

| Feature | SEARCH_PREVIEW (SQL) | Python API |
|---------|---------------------|------------|
| Use case | Testing and validation | Production applications |
| Input | String literals only | Programmatic parameters |
| Latency | Higher | Lower |
| Batch queries | No | Yes (in application code) |
| Recommended for apps | No | Yes |

### Can I use Cortex Search in a WHERE clause?

Not directly. Cortex Search is queried through SEARCH_PREVIEW (SQL), the Python API, or the REST API. It is not a SQL function that can be used inline in SELECT or WHERE clauses.

## Cost

### What does Cortex Search cost?

Cortex Search costs come from three areas:

1. **Serverless compute**: Index building and refresh (charged as Cortex Search credits)
2. **Warehouse compute**: The warehouse specified in the service is used for materializing the source query
3. **Storage**: The search index is stored in Snowflake

### How do I minimize cost?

1. **Increase TARGET_LAG**: Less frequent refreshes = lower compute cost
2. **Use a smaller warehouse**: For the source query materialization
3. **Limit data scope**: Filter the source query to relevant data
4. **Suspend unused services**: Stop refreshes when not needed
5. **Use multi-index queries**: Query specific indexes to reduce per-query cost

### Does querying cost credits?

Yes, each query consumes a small amount of Cortex Search compute credits. Cost scales with query volume and complexity.

## Performance

### What is the typical query latency?

- Python API / REST API: Typically 50-200ms
- SEARCH_PREVIEW: Higher (not optimized for production)

Latency depends on index size, query complexity, and filters.

### How long does service creation take?

Depends on data size and warehouse size:

| Data Size | Warehouse Size | Approximate Time |
|-----------|---------------|-----------------|
| <10K rows | X-Small | 1-5 minutes |
| 10K-100K rows | Small | 5-15 minutes |
| 100K-1M rows | Medium | 15-60 minutes |
| 1M+ rows | Medium-Large | 1+ hours |

### How can I speed up service creation?

1. Use a larger warehouse
2. Reduce data volume in the source query
3. Use `INITIALIZE = ON_SCHEDULE` to defer the first build

### Does Cortex Search scale to millions of rows?

Yes. Cortex Search is designed for large datasets. For very large tables:
- Use a Medium or Large warehouse for initial index build
- Set an appropriate TARGET_LAG
- Consider filtering the source query to the most relevant data

## Integration

### Can I use Cortex Search in Streamlit?

Yes. Use the Python SDK in Streamlit in Snowflake:

```python
from snowflake.core import Root
from snowflake.snowpark.context import get_active_session

session = get_active_session()
root = Root(session)
service = root.databases["DB"].schemas["SCHEMA"].cortex_search_services["SVC"]
resp = service.search(query="my query", columns=["col1"], limit=5)
```

### Can I use Cortex Search with Cortex Agents?

Yes. Cortex Search services can be used as tools within Cortex Agents. The agent can invoke the search service to retrieve relevant context before generating responses.

### Can I call it from a REST API?

Yes. Cortex Search exposes a REST endpoint:

```
POST https://<account_url>/api/v2/databases/<DB>/schemas/<SCHEMA>/cortex-search-services/<SERVICE>:query
```

Authenticate with a programmatic access token (PAT), JWT, or OAuth.

### Can I use it in dbt?

Cortex Search services are created and managed with SQL DDL, not dbt models. However, you can:
- Use dbt to prepare and maintain the source tables
- Create the service using a dbt `run-operation` or post-hook
- Reference the service in downstream queries

### Can I use it in a notebook?

Yes. Use SEARCH_PREVIEW in SQL cells or the Python API in Python cells:

```python
# In a Snowflake notebook Python cell
from snowflake.core import Root
from snowflake.snowpark.context import get_active_session

session = get_active_session()
root = Root(session)
service = root.databases["DB"].schemas["SCHEMA"].cortex_search_services["SVC"]
resp = service.search(query="test", columns=["col1"], limit=5)
resp.to_json()
```

## Technical Details

### What embedding models are supported?

| Model | Description |
|-------|-------------|
| `snowflake-arctic-embed-l-v2.0` | High-quality, 1024 dimensions |
| `snowflake-arctic-embed-m-v1.5` | Good quality, 768 dimensions |

You can also bring your own vector embeddings with multi-index services.

### What regions support Cortex Search?

Cortex Search is available in regions that support Cortex AI functions. Check the [Snowflake documentation on region availability](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions#availability) for the current list.

### What data types are supported for ATTRIBUTES?

ATTRIBUTES columns can be:
- VARCHAR / STRING
- NUMBER / INT / FLOAT
- DATE
- TIMESTAMP
- ARRAY (for @contains filters)
- BOOLEAN

### Can I search non-English text?

Yes. The Snowflake Arctic embedding models support multiple languages. Performance is best for common languages (English, Spanish, French, German, etc.) and may vary for less common languages.

### What is CORTEX_SEARCH_DATA_SCAN?

A table function that lets you inspect the contents of a search service's index:

```sql
SELECT * FROM TABLE(
    CORTEX_SEARCH_DATA_SCAN(SERVICE_NAME => 'my_service')
);
```

Useful for verifying that data was indexed correctly.

## Troubleshooting

### Why am I getting empty results?

- Service index may still be building (wait for creation to complete)
- Filter is too restrictive (try without filters)
- Data wasn't loaded into the source table
- Query doesn't match any content

### Why is my service not refreshing?

- Change tracking not enabled on the base table
- Warehouse is suspended
- Source query has errors
- Check TARGET_LAG setting

### Why is SEARCH_PREVIEW slow?

SEARCH_PREVIEW is designed for testing, not production. Use the Python SDK or REST API for low-latency queries.

### My Python SDK says "module not found"

Install or upgrade:
```bash
pip install snowflake -U
```

Requires version 0.8.0 or later.

---

## Getting the Latest Documentation

### Official Snowflake Documentation

| Topic | URL |
|-------|-----|
| Cortex Search Overview | https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-search/cortex-search-overview |
| CREATE CORTEX SEARCH SERVICE | https://docs.snowflake.com/en/sql-reference/sql/create-cortex-search |
| Query a Cortex Search Service | https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-search/query-cortex-search-service |
| SEARCH_PREVIEW Function | https://docs.snowflake.com/en/sql-reference/functions/search_preview-snowflake-cortex |
| Cortex AI Overview | https://docs.snowflake.com/en/guides-overview-ai-features |
| Supported Regions | https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions#availability |

### How to Fetch Latest Docs in Cortex Code

Ask the agent to fetch the latest documentation:

```
"Fetch the latest docs for Cortex Search from Snowflake"
"What's new in Cortex Search?"
"Get the current CREATE CORTEX SEARCH SERVICE syntax"
```

The agent can use `web_fetch` to retrieve the most current information directly from Snowflake's documentation site.

### Release Notes

| Resource | URL |
|----------|-----|
| Snowflake Release Notes | https://docs.snowflake.com/en/release-notes |
| Cortex AI Release Notes | https://docs.snowflake.com/en/release-notes/new-features#cortex-ai |
| BCR (Behavior Change) Bundles | https://docs.snowflake.com/en/release-notes/bcr-bundles |

### SNOWFLAKE_LEARNING Environment

| Resource | URL |
|----------|-----|
| Learning Environment (BCR-1992) | https://docs.snowflake.com/en/release-notes/bcr-bundles/un-bundled/bcr-1992 |
| Snowsight Templates | https://docs.snowflake.com/en/user-guide/ui-snowsight/snowsight-templates |

### Staying Current

1. **Bookmark the docs**: Save the Cortex Search overview page for quick reference
2. **Check release notes**: Review monthly for new features
3. **Ask in Cortex Code**: The agent can fetch latest docs on demand
4. **Snowflake Community**: https://community.snowflake.com for discussions and tips
