# Choosing Good Categories

## The Golden Rules

1. **Mutually Exclusive**: Each text should fit only ONE category
2. **Collectively Exhaustive**: Categories cover ALL possibilities
3. **Clearly Distinct**: Categories are obviously different from each other
4. **Appropriately Granular**: Not too broad, not too specific

## Number of Categories

### Minimum: 2 Categories

```python
categories = ["Positive", "Negative"]
```

For binary classification. Simple but forces a choice.

### Sweet Spot: 3-7 Categories

```python
categories = ["Positive", "Negative", "Neutral"]
# or
categories = ["Very Satisfied", "Satisfied", "Neutral", "Dissatisfied", "Very Dissatisfied"]
```

Optimal range for accuracy and usefulness.

### Maximum: 100 Categories

Supported but not recommended. Accuracy decreases significantly beyond 10 categories.

### Accuracy vs. Category Count

| Categories | Typical Accuracy | Use Case |
|------------|-----------------|----------|
| 2-3 | 90%+ | Simple decisions |
| 4-7 | 85-95% | Most business use cases |
| 8-15 | 70-85% | Complex taxonomies |
| 15+ | 50-70% | Consider hierarchical approach |

## Common Category Patterns

### Sentiment (3 categories)

```python
categories = ["Positive", "Negative", "Neutral"]
```

Best for general sentiment analysis.

### Sentiment with Intensity (5 categories)

```python
categories = ["Very Positive", "Positive", "Neutral", "Negative", "Very Negative"]
```

For nuanced sentiment tracking.

### Binary with Uncertainty (3 categories)

```python
categories = ["Yes", "No", "Unsure"]
# or
categories = ["Likely", "Unlikely", "Unsure"]
```

Best for recommendation or decision-based classification.

### Intent Detection (4-5 categories)

```python
categories = ["Question", "Complaint", "Praise", "Request", "Other"]
```

For understanding customer intent.

### Priority/Urgency (3-4 categories)

```python
categories = ["Critical", "High", "Medium", "Low"]
# or
categories = ["Urgent", "Normal", "Low"]
```

For ticket triage.

### Topic/Department Routing

```python
categories = ["Technical", "Billing", "Sales", "General"]
```

For directing inquiries to the right team.

## Category Design Mistakes

### Mistake 1: Overlapping Categories

❌ Bad:
```python
categories = ["Good", "Great", "Excellent", "Amazing"]  # All mean positive!
```

✅ Good:
```python
categories = ["Excellent", "Good", "Average", "Poor"]  # Distinct levels
```

### Mistake 2: Missing Coverage

❌ Bad:
```python
categories = ["Bug", "Feature Request"]  # What about questions?
```

✅ Good:
```python
categories = ["Bug", "Feature Request", "Question", "Other"]
```

### Mistake 3: Inconsistent Granularity

❌ Bad:
```python
categories = ["Electronics", "Clothing", "iPhone Cases", "Home Goods"]
# iPhone Cases is too specific compared to others
```

✅ Good:
```python
categories = ["Electronics", "Clothing", "Accessories", "Home Goods"]
```

### Mistake 4: Ambiguous Labels

❌ Bad:
```python
categories = ["Type A", "Type B", "Type C"]  # Meaningless to LLM
```

✅ Good:
```python
categories = ["Technical Issue", "Billing Question", "General Inquiry"]
```

### Mistake 5: No Catch-All

❌ Bad:
```python
categories = ["Pricing", "Features", "Integration"]
# What if the text is about something else?
```

✅ Good:
```python
categories = ["Pricing", "Features", "Integration", "Other"]
```

## The "Other/Unsure" Category

**Always include one.** It prevents:
- Forcing unclear text into wrong categories
- False confidence in edge cases
- Garbage-in, garbage-out classifications

### Options for Catch-All Categories

```python
"Other"           # Generic catch-all
"Unsure"          # For binary-style decisions
"Mixed"           # When text contains multiple sentiments
"Unclear"         # When text is ambiguous
"Not Applicable"  # When text doesn't fit the classification task
```

## Testing Your Categories

### Step 1: Sample Edge Cases

Write 5-10 edge case texts that might be ambiguous:

```python
edge_cases = [
    "It's okay, nothing special but does the job",  # Neutral edge
    "Great product but terrible shipping",  # Mixed
    "???",  # Unclear/garbage
    "Can someone help me?",  # Question, not sentiment
]
```

### Step 2: Run Classification

```python
for text in edge_cases:
    result = cortex.classify_text(text, categories, task_description)
    print(f"{result['label']} ({result['score']:.2f}): {text[:50]}...")
```

### Step 3: Evaluate Results

- Do edge cases get appropriate categories?
- Are confidence scores lower for ambiguous texts?
- Does "Other/Unsure" catch unclear inputs?

### Step 4: Refine Categories

Based on results, you might:
- Add a missing category
- Combine overlapping categories
- Clarify category names
- Adjust task description

## Domain-Specific Category Examples

### E-commerce Reviews

```python
categories = ["Would Recommend", "Would Not Recommend", "Undecided"]
```

### Customer Support Tickets

```python
categories = ["Bug Report", "Feature Request", "How-To Question", 
              "Account Issue", "Billing", "Other"]
```

### Social Media Monitoring

```python
categories = ["Positive Mention", "Negative Mention", 
              "Neutral Mention", "Not About Us"]
```

### Content Moderation

```python
categories = ["Safe", "Needs Review", "Violation"]
```

### Employee Feedback

```python
categories = ["Highly Engaged", "Engaged", "Neutral", 
              "Disengaged", "Highly Disengaged"]
```

### Sales Lead Qualification

```python
categories = ["Hot Lead", "Warm Lead", "Cold Lead", "Not a Lead"]
```

## Advanced: Hierarchical Categories

For complex taxonomies, classify in stages:

### Level 1: Broad Categories

```python
broad_categories = ["Technical", "Business", "Administrative"]
result1 = cortex.classify_text(text, broad_categories)
```

### Level 2: Specific Categories

```python
if result1['label'] == "Technical":
    specific = ["Bug", "Feature Request", "Integration", "Performance"]
elif result1['label'] == "Business":
    specific = ["Pricing", "Sales", "Partnership"]
# etc.
result2 = cortex.classify_text(text, specific)
```

This approach:
- Maintains high accuracy at each level
- Handles complex taxonomies (20+ final categories)
- Allows different task descriptions per level
