# Agent Evaluation: With Human Responses vs Without Human Responses

There are two common evaluation scenarios. In both cases, the same rubrics can be used. The main difference is what acts as the reference answer.

## Rubrics Used in the MVP

For the first MVP, use these two rubrics:

### Correctness

Measures whether the candidate agent response is factually consistent with the trusted reference.

Example 1–5 scale:

- **5** = Fully correct
- **4** = Mostly correct; minor issue
- **3** = Partially correct; noticeable issue
- **2** = Major factual issue
- **1** = Incorrect

### Completeness

Measures whether the candidate agent response includes all important information contained in the trusted reference.

Example 1–5 scale:

- **5** = Covers all important information
- **4** = Minor omission
- **3** = Important information missing
- **2** = Major parts missing
- **1** = Fails to cover the expected answer

The LLM Judge uses these rubric definitions to score each candidate response.

---

## 1. When Human Reference Responses Already Exist

### What You Have

- User query
- Human reference answer
- Agent response

### What You Are Doing

You are directly evaluating the quality of the agent response against the human reference answer.

### Flow

```text
User Query
+
Human Reference Answer
+
Agent Response
+
Rubrics
    ├── Correctness
    └── Completeness
        ↓
LLM Judge
        ↓
Structured Scores + Reasons
```

### How the Rubrics Are Used

#### Correctness

The Judge compares the agent response with the human reference and checks whether the information is factually correct.

#### Completeness

The Judge checks whether the agent response contains all important information present in the human reference.

### Example

```text
Question:
What documents are required?

Human Reference:
Aadhaar, PAN and address proof are required.

Agent Response:
Aadhaar and PAN are required.
```

Possible Judge result:

```json
{
  "correctness": {
    "score": 4,
    "reason": "The information provided is correct."
  },
  "completeness": {
    "score": 3,
    "reason": "Address proof from the reference answer is missing."
  }
}
```

### Purpose

This is primarily about **directly evaluating the agent's response quality**.

The human-written response acts as the reference or ground truth.

---

## 2. When Human Reference Responses Do Not Exist

### What You Have

- User query
- Agent V1 response
- SME/human review confirming that the Agent V1 response is 100% correct

There is no separately written human answer.

### What Happens Next

Because the SME has explicitly approved the Agent V1 response as correct, that response can be frozen and used as the **SME-approved reference response**.

Then the same question is run through Agent V2, or a newer version of the same agent.

### Flow

```text
User Query
        ↓
Agent V1 Response
        ↓
SME Reviews Response
        ↓
SME Confirms 100% Correct
        ↓
Freeze Agent V1 Response
as SME-Approved Reference
        ↓

Same User Query
        ↓
Agent V2
        ↓
Agent V2 Response
        ↓

User Query
+
SME-Approved Agent V1 Reference
+
Agent V2 Response
+
Rubrics
    ├── Correctness
    └── Completeness
        ↓
LLM Judge
        ↓
Structured Scores + Reasons
```

### How the Rubrics Are Used

#### Correctness

The Judge checks whether Agent V2's response is factually consistent with the SME-approved Agent V1 reference.

#### Completeness

The Judge checks whether Agent V2's response contains all important information present in the SME-approved reference.

### Example

```text
Question:
What documents are required?

SME-Approved Agent V1 Response:
Aadhaar, PAN and address proof are required.

Agent V2 Response:
Aadhaar and PAN are required.
```

Possible Judge result:

```json
{
  "correctness": {
    "score": 4,
    "reason": "The information provided is correct."
  },
  "completeness": {
    "score": 3,
    "reason": "Address proof from the SME-approved reference is missing."
  }
}
```

### Purpose

This is primarily about **evaluating Agent V2 against a benchmark created from SME-approved Agent V1 responses**.

It can also be used to compare versions of the same agent.

Important points:

- Agent V1 is not being evaluated again.
- Its SME-approved responses are acting as the benchmark.
- Agent V2 is the candidate being evaluated.
- Agent V2 does not have to be a completely different agent.
- It can be a newer version of the same agent after changes to the prompt, model, retrieval setup, tools, knowledge base, or other configuration.

---

## Summary

| Scenario | Trusted Reference | Candidate Being Evaluated | Rubrics |
|---|---|---|---|
| Human answers exist | Human-written reference answer | Agent response | Correctness, Completeness |
| Human answers do not exist | SME-approved Agent V1 response | Agent V2 response | Correctness, Completeness |

---

## Key Point

Rubrics are useful in **both scenarios**.

The difference is only the source of the trusted reference.

### Case 1

```text
Human-written answer
        ↓
Reference
```

### Case 2

```text
Agent V1 answer
+
SME approval
        ↓
SME-approved reference
```

In both cases, the LLM Judge evaluates the candidate response using the same rubric definitions.

---

## Common MVP Evaluation Flow

```text
Question
+
Trusted Reference
+
Candidate Agent Response
+
Correctness Rubric
+
Completeness Rubric
        ↓
LLM Judge
        ↓
Scores + Reasons
```

---

## What the Structured Judge Output Can Look Like

```json
{
  "correctness": {
    "score": 5,
    "reason": "The response is fully consistent with the trusted reference."
  },
  "completeness": {
    "score": 4,
    "reason": "The response covers the main answer but omits one minor detail."
  }
}
```

---

## Final Difference Between the Two Scenarios

### Scenario 1: Human Response Exists

```text
Question
+
Human Reference Answer
+
Agent Response
        ↓
LLM Judge
        ↓
Evaluate Agent Response
```

Here, the human answer already provides the trusted reference.

### Scenario 2: Human Response Does Not Exist

```text
Question
+
Agent V1 Response
+
SME Approval
        ↓
SME-Approved Reference

Same Question
        ↓
Agent V2 Response
        ↓
LLM Judge
        ↓
Evaluate Agent V2 Response
```

Here, the human does not write a separate answer.

Instead, the SME reviews the Agent V1 response and confirms that it is correct. That approved response is then used as the trusted reference for evaluating future agent responses.

---

## MVP Recommendation

For the initial MVP, keep the evaluation limited to:

- **Correctness**
- **Completeness**

Use a **1–5 scoring scale** and return a short explanation for every score.

The MVP therefore only needs to answer:

> Given a user question, a trusted reference response, and a candidate agent response, can an LLM Judge reliably score the candidate for correctness and completeness?

If this works well on the initial dataset, additional rubrics, human calibration, modular configuration, larger datasets, and production evaluation can be added later.