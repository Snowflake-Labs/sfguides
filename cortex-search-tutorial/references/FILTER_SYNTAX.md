# Filter Syntax Reference

Cortex Search supports filtering on columns declared as `ATTRIBUTES` in the `CREATE CORTEX SEARCH SERVICE` statement. Filters narrow search results before they are returned.

## Filter Operators

### @eq — Equality

Matches rows where a column equals an exact value. Works with strings and numbers.

**String equality:**
```json
{"@eq": {"region": "North America"}}
```

**Numeric equality:**
```json
{"@eq": {"priority_level": 3}}
```

**SQL example:**
```sql
SELECT PARSE_JSON(
    SNOWFLAKE.CORTEX.SEARCH_PREVIEW(
        'my_service',
        '{
            "query": "billing help",
            "columns": ["transcript_text", "region"],
            "filter": {"@eq": {"region": "Europe"}},
            "limit": 5
        }'
    )
)['results'] AS results;
```

**Python example:**
```python
resp = service.search(
    query="billing help",
    columns=["transcript_text", "region"],
    filter={"@eq": {"region": "Europe"}},
    limit=5
)
```

### @contains — Array Contains

Matches rows where an array column contains a specified value.

```json
{"@contains": {"tags": "urgent"}}
```

Useful when a column stores arrays of values (e.g., tags, categories, labels).

### @gte — Greater Than or Equal To

Matches rows where a numeric or date/timestamp column is greater than or equal to a value.

**Numeric:**
```json
{"@gte": {"score": 0.8}}
```

**Date:**
```json
{"@gte": {"created_date": "2024-11-10"}}
```

**Timestamp:**
```json
{"@gte": {"created_at": "2024-11-10T00:00:00.000+00:00"}}
```

### @lte — Less Than or Equal To

Matches rows where a numeric or date/timestamp column is less than or equal to a value.

**Numeric:**
```json
{"@lte": {"score": 0.5}}
```

**Date:**
```json
{"@lte": {"created_date": "2024-11-15"}}
```

### @primarykey — Primary Key Equality

Matches a specific row by its primary key. Only available for services configured with a primary key.

```json
{"@primarykey": {"transcript_id": 1005}}
```

For compound primary keys, provide all key columns:

```json
{"@primarykey": {"order_id": 100, "line_id": 3}}
```

## Logical Operators

### @and — All Conditions Must Match

Combines multiple filters where ALL must be true.

```json
{
    "@and": [
        {"@eq": {"region": "North America"}},
        {"@eq": {"product_category": "Billing"}}
    ]
}
```

### @or — Any Condition Can Match

Combines multiple filters where ANY can be true.

```json
{
    "@or": [
        {"@eq": {"region": "North America"}},
        {"@eq": {"region": "Europe"}}
    ]
}
```

### @not — Negate a Condition

Inverts a filter condition.

```json
{
    "@not": {"@eq": {"region": "Asia"}}
}
```

## Combining Operators

Operators can be nested to build complex filter logic.

### Example: Region is North America AND category is NOT Hardware

```json
{
    "@and": [
        {"@eq": {"region": "North America"}},
        {"@not": {"@eq": {"product_category": "Hardware"}}}
    ]
}
```

### Example: Date range (between two dates)

```json
{
    "@and": [
        {"@gte": {"created_date": "2024-11-05"}},
        {"@lte": {"created_date": "2024-11-15"}}
    ]
}
```

### Example: (North America OR Europe) AND Billing

```json
{
    "@and": [
        {
            "@or": [
                {"@eq": {"region": "North America"}},
                {"@eq": {"region": "Europe"}}
            ]
        },
        {"@eq": {"product_category": "Billing"}}
    ]
}
```

### Example: Exclude multiple values

```json
{
    "@and": [
        {"@not": {"@eq": {"product_category": "Hardware"}}},
        {"@not": {"@eq": {"product_category": "Account"}}}
    ]
}
```

## Common Filter Patterns

### Filter by single attribute

```python
# Python
resp = service.search(
    query="internet issues",
    columns=["transcript_text", "region"],
    filter={"@eq": {"region": "North America"}},
    limit=5
)
```

```sql
-- SQL
'{"query": "internet issues", "columns": ["transcript_text", "region"], "filter": {"@eq": {"region": "North America"}}, "limit": 5}'
```

### Filter by multiple attributes (AND)

```python
resp = service.search(
    query="slow speeds",
    columns=["transcript_text"],
    filter={
        "@and": [
            {"@eq": {"region": "Europe"}},
            {"@eq": {"product_category": "Internet"}}
        ]
    },
    limit=5
)
```

### Filter by date range

```python
resp = service.search(
    query="refund request",
    columns=["transcript_text", "created_date"],
    filter={
        "@and": [
            {"@gte": {"created_date": "2024-11-01"}},
            {"@lte": {"created_date": "2024-11-10"}}
        ]
    },
    limit=5
)
```

### Filter with OR (multiple possible values)

```python
resp = service.search(
    query="account help",
    columns=["transcript_text", "product_category"],
    filter={
        "@or": [
            {"@eq": {"product_category": "Account"}},
            {"@eq": {"product_category": "Billing"}}
        ]
    },
    limit=5
)
```

### No filter (search everything)

```python
# Simply omit the filter parameter
resp = service.search(
    query="customer complaint",
    columns=["transcript_text", "region"],
    limit=5
)
```

## Usage Notes

- Filters only work on columns declared in the `ATTRIBUTES` clause of `CREATE CORTEX SEARCH SERVICE`.
- `@eq` on strings is case-sensitive.
- `@gte` and `@lte` do not work with fixed-point numeric values over 19 digits.
- `@contains` works with array-type columns.
- Timestamp filters accept the format: `YYYY-MM-DDTHH:MM:SS.sss+HH:MM`. If timezone is omitted, UTC is assumed.
- Date filters accept the format: `YYYY-MM-DD`. Time and timezone portions are truncated.
- Matching against `NaN` values follows Snowflake's special value handling.
- Filters are applied after the search ranking, narrowing the result set.

## Filter Syntax Errors

### Common Mistakes

**Wrong: Missing @ prefix**
```json
{"eq": {"region": "North America"}}
```

**Correct:**
```json
{"@eq": {"region": "North America"}}
```

**Wrong: Using @and with an object instead of array**
```json
{"@and": {"@eq": {"region": "North America"}, "@eq": {"product_category": "Billing"}}}
```

**Correct: @and takes an array**
```json
{"@and": [{"@eq": {"region": "North America"}}, {"@eq": {"product_category": "Billing"}}]}
```

**Wrong: Filtering on a non-ATTRIBUTES column**
```json
{"@eq": {"agent_id": "AG2001"}}
```
This fails if `agent_id` was not listed in the `ATTRIBUTES` clause. Only ATTRIBUTES columns can be filtered.

**Wrong: Using @contains on a non-array column**
```json
{"@contains": {"region": "North"}}
```
`@contains` is for array columns. For substring matching on strings, Cortex Search does not support it in filters — use the search query itself instead.
