# Frequently Asked Questions

## Basic Questions

### What is CLASSIFY_TEXT?

CLASSIFY_TEXT is a Snowflake Cortex function that uses AI to categorize text into user-defined categories. Give it any text and a list of categories, and it returns the best matching category.

### Do I need to train a model?

No. CLASSIFY_TEXT uses zero-shot classification - it works immediately with any categories you provide. No training data, no model training, no deployment.

### What's the difference between CLASSIFY_TEXT and SENTIMENT?

| Function | Use Case | Output |
|----------|----------|--------|
| CLASSIFY_TEXT | Custom categories you define | Any category label you specify |
| SENTIMENT | General sentiment only | positive, negative, neutral + score |

Use CLASSIFY_TEXT when you need custom categories beyond positive/negative/neutral.

### How accurate is it?

Accuracy depends on:
- **Category design**: Clear, distinct categories → higher accuracy
- **Task description**: Specific context → higher accuracy
- **Text quality**: Clear text → higher accuracy

Typical accuracy ranges from 70-95% for well-designed categories.

### How much does it cost?

CLASSIFY_TEXT consumes Cortex AI credits based on:
- Input text length (tokens)
- Categories (small token cost)
- Task description (if provided)

For pricing details, see Snowflake's Cortex pricing documentation.

## Categories

### How many categories can I use?

- Minimum: 2 categories
- Maximum: 100 categories
- Recommended: 3-10 categories for best accuracy

### Should I include an "Other" category?

Yes, almost always. Without a catch-all category, ambiguous text is forced into inappropriate categories. Options:
- "Other"
- "Unsure"
- "Not Applicable"
- "Unknown"

### Can I use the same text for multiple categories?

CLASSIFY_TEXT returns ONE category. For multi-label classification, run CLASSIFY_TEXT multiple times with different category sets, or use a hierarchical approach.

### What makes good categories?

1. **Mutually exclusive**: Text fits only one category
2. **Clearly distinct**: Categories are obviously different
3. **Exhaustive**: All possibilities are covered
4. **Balanced**: Similar level of specificity

## Task Descriptions

### Are task descriptions required?

No, but they significantly improve accuracy by telling the LLM what aspect of the text to classify.

### How long should a task description be?

Aim for 30-50 words. Enough to provide context, but not so long it adds significant cost or causes confusion.

### Can task descriptions hurt accuracy?

Yes, if they're:
- Confusing or contradictory
- Too long and rambling
- Include biasing examples

Keep them clear and concise.

## Python vs SQL

### When should I use Python?

- Interactive exploration and prototyping
- Complex preprocessing logic
- Integration with ML pipelines
- When you need both label and score easily

### When should I use SQL?

- Production pipelines
- Creating views
- Scheduled queries
- Teams without Python expertise
- Simple transformations

### Is one faster than the other?

No significant difference for the classification itself. Both parallelize within Snowflake. Python has slight overhead for session setup on small queries.

## Technical Details

### What model powers CLASSIFY_TEXT?

CLASSIFY_TEXT uses Snowflake's Cortex LLM infrastructure. The specific model may vary but is optimized for classification tasks.

### What's the maximum text length?

Text should be under ~4000 tokens (roughly 3000-4000 words). For longer text, truncate or summarize first.

### Can I classify non-English text?

Yes, Cortex LLMs are multilingual. Performance is best for common languages (English, Spanish, French, German, etc.) and may vary for less common languages.

### Does CLASSIFY_TEXT work with structured data?

It's designed for unstructured text. For structured data, traditional SQL logic or ML models are usually better.

## Performance

### How long does classification take?

- Single text: ~200-500ms
- 1000 rows: ~2-5 seconds
- 10,000 rows: ~20-60 seconds

Times vary based on text length, warehouse size, and system load.

### How can I speed up classification?

1. Use a larger warehouse
2. Process in batches
3. Classify only new/changed rows
4. Cache results (don't re-classify same text)

### Does it scale to millions of rows?

Yes, but:
- Process in batches
- Consider incremental processing
- Monitor costs
- Use appropriate warehouse size

## Best Practices

### Should I store classification results?

Yes, for:
- Avoiding re-classification costs
- Faster queries
- Historical analysis
- Audit trails

No, if:
- Text changes frequently
- Always need fresh classification
- Storage is more expensive than re-computation

### How do I handle classification errors?

1. Add logging to track errors
2. Use confidence scores to flag uncertain results
3. Have a human review queue for low-confidence classifications
4. Retry transient failures with exponential backoff

### Can I customize the model?

CLASSIFY_TEXT uses Snowflake-managed models. For custom models:
- Use Snowpark ML to train your own
- Consider COMPLETE with custom prompts
- Deploy custom models via Model Registry

### How do I validate classification quality?

1. Test on a labeled sample dataset
2. Review edge cases manually
3. Monitor confidence score distribution
4. Track accuracy over time
5. Collect user feedback on classifications

## Troubleshooting

### Why am I getting NULL results?

- Input text is NULL or empty
- Categories array is invalid
- Error during classification (check Query History)

### Why are all my results the same category?

- Categories are too similar
- Task description is biasing toward one category
- Text is uniformly similar
- Need to check sample of your data

### Why is my confidence score always low?

- Text is genuinely ambiguous
- Categories don't fit the text well
- Missing task description
- Too many similar categories

### My query is timing out

- Use larger warehouse
- Process in smaller batches
- Check for expensive preprocessing
- Increase session timeout

## Integration

### Can I use CLASSIFY_TEXT in dbt?

Yes, just use the SQL syntax in your dbt models:
```sql
SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(...) ...
```

### Can I use it in Streamlit?

Yes, use the Python Snowpark syntax:
```python
import snowflake.cortex as cortex
result = cortex.classify_text(text, categories)
```

### Can I use it in BI tools?

Yes, create a view that includes classified columns, then connect your BI tool to that view.

### Can I call it from an API?

Use Snowflake's SQL API or Snowpark Python to call CLASSIFY_TEXT programmatically from external applications.

---

## Getting the Latest Documentation

### Official Snowflake Documentation

The most up-to-date information is always in the official Snowflake docs:

| Topic | URL |
|-------|-----|
| CLASSIFY_TEXT Function | https://docs.snowflake.com/en/sql-reference/functions/classify_text-snowflake-cortex |
| Cortex LLM Functions Overview | https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions |
| Cortex AI Overview | https://docs.snowflake.com/en/guides-overview-ai-features |
| Supported Regions | https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions#availability |

### How to Fetch Latest Docs in Cortex Code

Ask the agent to fetch the latest documentation:

```
"Fetch the latest docs for CLASSIFY_TEXT from Snowflake"
"What's new in Cortex AI functions?"
"Get the current documentation for cortex.classify_text"
```

The agent can use `web_fetch` to retrieve the most current information directly from Snowflake's documentation site.

### Release Notes

Check for new features and changes:

| Resource | URL |
|----------|-----|
| Snowflake Release Notes | https://docs.snowflake.com/en/release-notes |
| Cortex AI Release Notes | https://docs.snowflake.com/en/release-notes/new-features#cortex-ai |
| BCR (Behavior Change) Bundles | https://docs.snowflake.com/en/release-notes/bcr-bundles |

### SNOWFLAKE_LEARNING Environment

For the learning environment used in this tutorial:

| Resource | URL |
|----------|-----|
| Learning Environment (BCR-1992) | https://docs.snowflake.com/en/release-notes/bcr-bundles/un-bundled/bcr-1992 |
| Snowsight Templates | https://docs.snowflake.com/en/user-guide/ui-snowsight/snowsight-templates |

### Staying Current

1. **Bookmark the docs**: Save the CLASSIFY_TEXT page for quick reference
2. **Check release notes**: Review monthly for new features
3. **Ask in Cortex Code**: The agent can fetch latest docs on demand
4. **Snowflake Community**: https://community.snowflake.com for discussions and tips
