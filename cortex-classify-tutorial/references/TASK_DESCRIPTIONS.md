# Writing Effective Task Descriptions

## Why Task Descriptions Matter

Task descriptions are **the most powerful lever** for improving CLASSIFY_TEXT accuracy. They tell the LLM exactly what aspect of the text to focus on.

## The Problem Without Task Descriptions

Consider this review:
```
"The delivery was 3 days late, but the product quality exceeded my expectations. 
The packaging was damaged, but everything inside was perfect."
```

Without a task description, classifying as "Positive" or "Negative":
- **Delivery experience?** → Negative (late, damaged packaging)
- **Product quality?** → Positive (exceeded expectations, perfect)
- **Overall?** → Ambiguous

The LLM doesn't know which aspect you care about.

## Anatomy of a Good Task Description

### Structure

```
[Context about the domain]
[What you want to classify]
[What each category means (optional)]
```

### Examples

**Simple:**
```
Classify customer sentiment about our food truck service.
```

**With Context:**
```
Tasty Bytes operates food trucks. Based on this customer review, 
determine if they would recommend our trucks to friends and family.
```

**With Category Definitions:**
```
Classify this support ticket priority:
- Critical: Production system down, data loss, or security breach
- High: Major feature broken, significant user impact
- Medium: Minor bug, workaround available
- Low: Cosmetic issues, feature requests
```

## Task Description Templates

### Sentiment Analysis

```python
task = """
Analyze the sentiment of this customer feedback about [PRODUCT/SERVICE].
Focus on their overall satisfaction, not individual complaints.
"""
```

### Recommendation Likelihood

```python
task = """
Based on this [review/feedback], determine if the customer would 
recommend [PRODUCT/SERVICE] to friends and family.
Consider both explicit recommendations and implied satisfaction.
"""
```

### Intent Classification

```python
task = """
Determine the customer's primary intent:
- Question: Seeking information or clarification
- Issue: Reporting a problem or complaint
- Request: Asking for a new feature or capability
- Feedback: Providing general comments or suggestions
"""
```

### Topic/Department Routing

```python
task = """
Route this support message to the appropriate department:
- Technical: Software bugs, integrations, API issues
- Billing: Payments, invoices, subscriptions, pricing
- Account: Login, permissions, user management
- General: All other inquiries
"""
```

### Urgency Classification

```python
task = """
Assess the urgency of this support request:
- Urgent: Production impact, data loss risk, time-sensitive
- Normal: Standard issues, no immediate business impact
- Low: Nice-to-have, cosmetic, future consideration
"""
```

### Compliance/Risk Assessment

```python
task = """
Evaluate if this content violates community guidelines:
- Compliant: Follows all guidelines
- Warning: Minor issues, may need editing
- Violation: Clear policy breach, requires action
"""
```

## Do's and Don'ts

### DO: Be Specific

✅ Good:
```
Based on this restaurant review, classify the diner's satisfaction 
with the FOOD QUALITY (taste, freshness, portion size).
```

❌ Vague:
```
Classify this review.
```

### DO: Define Ambiguous Categories

✅ Good:
```
Categories:
- Likely: Explicit recommendation or strong positive language
- Unlikely: Explicit advice against or strong negative language  
- Unsure: Mixed signals, neutral tone, or insufficient information
```

❌ Undefined:
```
["Likely", "Unlikely", "Unsure"]  # LLM must guess what these mean
```

### DO: Specify What to Ignore

✅ Good:
```
Classify satisfaction with the PRODUCT. Ignore shipping delays or 
packaging issues - focus only on the product itself.
```

❌ Ambiguous:
```
Classify satisfaction.  # Which aspect?
```

### DON'T: Include Examples in Task Description

❌ Avoid:
```
Classify sentiment. For example, "I love it" is Positive, 
"I hate it" is Negative.  # Can bias the model
```

✅ Better:
```
Classify the overall sentiment expressed in this text.
```

### DON'T: Make It Too Long

❌ Avoid:
```
This is a customer review from our e-commerce platform where customers 
purchase various products including electronics, clothing, home goods, 
and accessories. We want to understand customer satisfaction levels to 
improve our services and product offerings. Please analyze the following 
review and determine if the customer is satisfied, dissatisfied, or 
neutral based on their comments about the product quality, shipping 
experience, customer service interaction, and overall value for money...
[200 more words]
```

✅ Better:
```
Classify this e-commerce review by overall customer satisfaction.
Focus on their feelings about the product received.
```

## Task Descriptions for Different Domains

### E-commerce

```python
task = """
Classify this product review:
- Recommend: Customer expresses satisfaction and would buy again
- Not Recommend: Customer expresses dissatisfaction or regret
- Neutral: Mixed feelings or insufficient opinion
"""
```

### Customer Support

```python
task = """
Categorize this support ticket by issue type:
- Technical: Bugs, errors, integration problems
- Account: Login, access, permissions
- Billing: Charges, refunds, subscriptions
- How-To: Feature questions, usage guidance
"""
```

### Social Media

```python
task = """
Classify the sentiment of this social media post about our brand:
- Positive: Praise, recommendation, satisfaction
- Negative: Complaint, criticism, frustration
- Neutral: Factual mention, question, or unrelated
"""
```

### HR/Feedback

```python
task = """
Analyze this employee feedback survey response:
- Engaged: Positive about work, team, company
- Disengaged: Frustrated, dissatisfied, considering leaving
- Neutral: Professional but not particularly positive or negative
"""
```

### Healthcare

```python
task = """
Classify this patient feedback about their visit:
- Satisfied: Good experience, would return
- Unsatisfied: Problems with care, wait time, or staff
- Neutral: No strong opinion expressed
"""
```

## Iterating on Task Descriptions

### Step 1: Start Simple

```python
task = "Classify customer sentiment."
```

### Step 2: Test on Sample Data

```python
# Run on 10-20 examples
results = df.limit(20).withColumn('sentiment', 
    cortex.classify_text(F.col('text'), categories, task))
results.show()
```

### Step 3: Review Errors

Look for patterns in misclassifications:
- Are mixed reviews being classified randomly?
- Are certain topics being misunderstood?
- Is the model focusing on the wrong aspect?

### Step 4: Refine Based on Errors

```python
# Before: Generic
task = "Classify customer sentiment."

# After: Specific based on observed errors
task = """
Classify customer sentiment about their ORDER EXPERIENCE.
Focus on satisfaction with what they received, not delivery timing.
"""
```

### Step 5: Validate Improvement

Re-run on the same samples and compare accuracy.

## Token Efficiency

Task descriptions add to token count (and cost). Keep them concise but complete.

**Tokens ≈ words × 1.3** (rough estimate)

| Description Length | Approximate Tokens | Per 1000 Calls |
|-------------------|-------------------|----------------|
| 20 words | ~26 tokens | +26K tokens |
| 50 words | ~65 tokens | +65K tokens |
| 100 words | ~130 tokens | +130K tokens |

Aim for 30-50 words for most use cases.
