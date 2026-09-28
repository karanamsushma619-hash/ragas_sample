# Evaluation Framework Design

## 1. Objective

The framework should be reusable and capable of integrating with different AI agents.

It should support different agent response formats, different LLM judges, configurable rubrics, and different types of trusted reference responses.

---

## 2. Configurable Components

### LLM Judge Configuration

- **LLM Model**
  - Model/provider used for evaluation.

- **LLM Temperature**
  - Recommended to keep low for consistent scoring.

- **Timeout**
  - Maximum time allowed for an evaluation call.

- **Max Retries**
  - Number of retries when the judge call fails.

---

### Rubric Configuration

Rubrics should be configurable depending on the agent being evaluated.

For the MVP:

- **Correctness**
- **Completeness**

Additional rubrics can be added later.

#### Correctness

Measures whether the candidate agent response is factually consistent with the trusted reference response.

Example scoring:

- **5** = Fully correct
- **4** = Mostly correct; minor issue
- **3** = Partially correct
- **2** = Major factual issue
- **1** = Incorrect

#### Completeness

Measures whether the candidate response contains all important information present in the trusted reference response.

Example scoring:

- **5** = Covers all important information
- **4** = Minor omission
- **3** = Important information missing
- **2** = Major parts missing
- **1** = Fails to cover the expected answer

---

## 3. Scoring Configuration

The scoring mechanism should also be configurable.

Example:

```text
Scale: 1–5

The meaning of each score should be defined as part of the rubric configuration.


---

4. Agent Response Configuration

Different agents may return responses in different formats.

Supported formats can include:

JSON — recommended

Markdown — required fields need to be extracted

Plain Text — required fields need to be extracted


Response Mapping

The framework should allow configuration of where the actual final answer exists.

Example:

{
  "answer": "..."
}

or:

{
  "response": {
    "content": "..."
  }
}

The framework should extract the candidate answer before sending it to the LLM Judge.


---

5. Configurable Reference Response Type

The framework should support two ways of providing the trusted reference response.

A configuration can determine which flow is being used.

Example:

reference_type: human_reference

or:

reference_type: sme_approved_agent_response


---

6. Flow 1 — Human Reference Response

Use this flow when a human-written correct response already exists.

Input

User Query

Human Reference Response

Candidate Agent Response


Flow

User Query
+
Human Reference Response
+
Candidate Agent Response
+
Configured Rubrics
        ↓
LLM Judge
        ↓
Correctness Score
Completeness Score
Reasons

The human response acts as the trusted reference / ground truth.

The LLM Judge evaluates the candidate agent response against this reference.


---

7. Flow 2 — SME-Approved Agent Response

Use this flow when there is no separately written human reference response.

Existing Data

User Query

Agent V1 Response

SME/Human Reviewer Score = 100% Correct


Because an SME has reviewed and approved the response, the Agent V1 response can be promoted to an:

SME-Approved Reference Response

Reference Creation Flow

User Query
        ↓
Agent V1 Response
        ↓
SME Reviews Response
        ↓
SME Confirms Response is Correct
        ↓
Freeze Response
        ↓
SME-Approved Reference Response

The same user query is then run against the current/new version of the agent.

Same User Query
        ↓
Agent V2 / Current Agent
        ↓
Candidate Agent Response

The evaluation becomes:

User Query
+
SME-Approved Agent V1 Reference
+
Candidate Agent V2 Response
+
Configured Rubrics
        ↓
LLM Judge
        ↓
Correctness Score
Completeness Score
Reasons

Agent V1 is not being evaluated again.

Its SME-approved response is acting as the trusted benchmark.

The candidate/current agent response is the response being evaluated.


---

8. Common Evaluation Function

Both flows should ultimately use the same evaluation function.

Conceptually:

evaluate_response(
    user_query,
    reference_response,
    candidate_response,
    rubrics
)

The evaluation function does not need to know whether the reference response came from a human or from an SME-approved agent response.

That distinction is handled through configuration.


---

9. Common LLM Judge Input

Regardless of the reference type, the LLM Judge receives:

User Query

Trusted Reference Response

Candidate Agent Response

Rubric Definitions

Scoring Definitions

The LLM Judge then compares the meaning, not exact wording.


---

10. Expected Structured Output

Example:

{
  "correctness": {
    "score": 5,
    "reason": "The candidate response is fully consistent with the trusted reference."
  },
  "completeness": {
    "score": 4,
    "reason": "The main information is covered, but one minor detail is missing."
  }
}


---

11. Overall Configurable Flow

Configuration
                     │
          ┌──────────┴──────────┐
          │                     │
 Human Reference        SME-Approved Agent
      Flow                 Response Flow
          │                     │
          └──────────┬──────────┘
                     ↓
             Trusted Reference
                     ↓

User Query
+
Trusted Reference
+
Candidate Agent Response
+
Configured Rubrics
        ↓
LLM Judge
        ↓
Structured Scores + Reasons


---

12. MVP Configuration Summary

The MVP should support configuration for:

LLM Model

LLM Temperature

Rubrics

Rubric Definitions

Scoring Scale

Response Format

Response Field Mapping

Reference Type

human_reference

sme_approved_agent_response


Max Retries

Timeout



---

13. MVP Goal

The MVP should answer one simple question:

> Given a user query, a trusted reference response, a candidate agent response, and configured rubrics, can the LLM Judge reliably produce meaningful correctness and completeness scores with explanations?



The trusted reference can come from either:

1. A human-written reference response, or


2. An SME-approved agent response.


