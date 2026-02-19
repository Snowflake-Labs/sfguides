# Cortex CLASSIFY_TEXT Deep Dive

## What is CLASSIFY_TEXT?

CLASSIFY_TEXT is a Snowflake Cortex LLM function that performs **zero-shot text classification**. Given any text and a list of categories, it determines which category best describes the text.

### Zero-Shot Classification Explained

Traditional machine learning classification requires:
1. Labeled training data (thousands of examples)
2. Model training (hours to days)
3. Model deployment and maintenance

Zero-shot classification (like CLASSIFY_TEXT):
1. No training data needed
2. Works immediately with any categories
3. Uses LLM's language understanding

**The LLM understands language well enough to classify text into categories it's never been explicitly trained on.**

## How It Works Under the Hood

```
┌──────────────────────────────────────────────────────────┐
│                    Your Text Input                        │
│  "This product is amazing! Best purchase ever!"          │
└──────────────────────────┬───────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────┐
│              Cortex LLM (Arctic/Llama/etc)               │
│                                                          │
│  Receives:                                               │
│  - Your text                                             │
│  - Categories: ["Positive", "Negative", "Neutral"]       │
│  - Task description (optional context)                   │
│                                                          │
│  Processes:                                              │
│  - Understands text meaning                              │
│  - Compares against each category                        │
│  - Calculates probability for each                       │
└──────────────────────────┬───────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────┐
│                    JSON Response                          │
│  {                                                       │
│    "label": "Positive",                                  │
│    "score": 0.95                                         │
│  }                                                       │
└──────────────────────────────────────────────────────────┘
```

## Function Signatures

### Python (Snowpark)

```python
snowflake.cortex.classify_text(
    text: str | Column,
    categories: List[str],
    task_description: str = None
) -> dict | Column
```

**Parameters:**
- `text`: The text to classify (string or DataFrame column)
- `categories`: List of possible categories (2-100 categories)
- `task_description`: Optional context explaining what to classify

**Returns:**
- `dict` with `label` (string) and `score` (float 0-1)
- For columns, returns a variant column

### SQL

```sql
SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
    text VARCHAR,
    categories ARRAY,
    options OBJECT
)
```

**Parameters:**
- `text`: VARCHAR column or literal
- `categories`: ARRAY of category strings
- `options`: OBJECT with optional `task_description` key

**Returns:**
- VARIANT containing JSON with `label` and `score`

## The Score: Understanding Confidence

The `score` field (0.0 to 1.0) represents the LLM's confidence:

| Score Range | Meaning | Action |
|-------------|---------|--------|
| 0.90 - 1.00 | Very confident | Trust the result |
| 0.70 - 0.89 | Confident | Generally reliable |
| 0.50 - 0.69 | Moderate | Review edge cases |
| 0.30 - 0.49 | Low confidence | Consider "Unsure" category |
| 0.00 - 0.29 | Very uncertain | Text may be ambiguous |

### Using Score in Queries

```sql
-- Only trust high-confidence classifications
SELECT 
  review,
  classification:label::STRING AS label,
  classification:score::FLOAT AS confidence
FROM (
  SELECT 
    review,
    SNOWFLAKE.CORTEX.CLASSIFY_TEXT(review, ['Positive', 'Negative']) AS classification
  FROM reviews
)
WHERE classification:score > 0.7;
```

## Category Design Principles

### Number of Categories

- **Minimum**: 2 categories
- **Maximum**: 100 categories (but accuracy decreases with more)
- **Optimal**: 3-10 categories for best results

### Good Categories

1. **Mutually exclusive**: A text should fit only one category
2. **Collectively exhaustive**: Categories cover all possibilities
3. **Distinct**: Categories are clearly different
4. **Balanced granularity**: Similar level of specificity

**Good Example:**
```python
categories = ["Positive", "Negative", "Neutral"]
```

**Bad Example:**
```python
categories = ["Excellent", "Great", "Good", "Okay", "Bad"]  # Excellent/Great/Good overlap
```

### Include "Other" or "Unsure"

Always include a catch-all category:

```python
categories = ["Bug Report", "Feature Request", "Question", "Other"]
```

This prevents forcing unclear text into wrong categories.

## Task Description Best Practices

The task description tells the LLM **what question you're asking**.

### Without Task Description

```python
# Ambiguous - classify based on what?
cortex.classify_text(
    "The package arrived late but the product works great",
    ["Positive", "Negative"]
)
# Could go either way - delivery negative, product positive
```

### With Task Description

```python
# Clear - classify based on product satisfaction
cortex.classify_text(
    "The package arrived late but the product works great",
    ["Positive", "Negative"],
    task_description="Classify based on the customer's satisfaction with the PRODUCT quality"
)
# Result: Positive (focuses on product, not delivery)
```

### Task Description Templates

**Sentiment Analysis:**
```
"Classify the sentiment of this customer feedback about our service."
```

**Intent Detection:**
```
"Determine the customer's intent: are they asking a question, reporting an issue, or requesting a feature?"
```

**Topic Classification:**
```
"Categorize this support ticket by department: Sales, Technical Support, Billing, or General Inquiry."
```

**Recommendation Likelihood:**
```
"Based on this review, would the customer recommend our product to others?"
```

## Performance Characteristics

### Latency

- Single text: 200-500ms typical
- Batch (DataFrame): Parallelized automatically
- SQL queries: Use appropriate LIMIT for testing

### Cost

- Charged as Cortex LLM credits
- Based on tokens processed (input text + categories)
- Task description adds to token count

### Throughput Tips

```python
# Good: Process entire column at once
df.withColumn('category', cortex.classify_text(F.col('text'), categories))

# Bad: Process row by row
for row in rows:  # Don't do this
    result = cortex.classify_text(row.text, categories)
```

## Comparison with Other Methods

| Approach | Training Required | Accuracy | Flexibility | Cost |
|----------|-------------------|----------|-------------|------|
| CLASSIFY_TEXT | None | Good-Excellent | Very High | Per-call |
| ML Model | Yes (lots of data) | Can be higher | Fixed categories | Training + inference |
| Rule-based | No | Limited | Medium | Minimal |
| SENTIMENT | None | Good | Fixed (pos/neg/neu) | Per-call |

## When to Use CLASSIFY_TEXT

**Use CLASSIFY_TEXT when:**
- You need custom categories
- You don't have labeled training data
- Categories might change
- Rapid prototyping
- Moderate volume (<millions of rows)

**Consider alternatives when:**
- Very high volume (millions daily)
- Fixed, well-defined categories with lots of training data
- Need guaranteed latency <100ms
- Simple positive/negative sentiment (use SENTIMENT instead)

## Advanced Patterns

### Hierarchical Classification

```python
# First: Broad category
broad = cortex.classify_text(text, ["Support", "Sales", "General"])

# Then: Specific subcategory
if broad['label'] == "Support":
    specific = cortex.classify_text(text, ["Technical", "Billing", "Account"])
```

### Multi-Label Classification

CLASSIFY_TEXT returns one label. For multi-label, run multiple times:

```python
# Is it urgent?
urgency = cortex.classify_text(text, ["Urgent", "Normal"])

# What topic?
topic = cortex.classify_text(text, ["Billing", "Technical", "General"])

# Combine results
result = f"{urgency['label']} {topic['label']}"  # "Urgent Technical"
```

### Confidence-Based Routing

```python
result = cortex.classify_text(text, categories)

if result['score'] > 0.8:
    # Auto-process with confidence
    process_automatically(result['label'])
elif result['score'] > 0.5:
    # Flag for human review
    queue_for_review(text, result)
else:
    # Escalate ambiguous cases
    escalate(text)
```
