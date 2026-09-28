Evaluation Framework Design
===========================

1. Framework Objective
----------------------

The evaluation framework should be generic and configurable so that it can be integrated with different AI agents without changing the core evaluation logic.


2. Configurable Components
--------------------------

LLM Judge Configuration
    - LLM Provider / Model
    - Temperature
    - Timeout
    - Maximum Retries

Evaluation Rubrics
    - Correctness
    - Completeness
    - Additional rubrics can be added later

Scoring Configuration
    - Score scale, for example: 1–5
    - Definition of each score
    - Rubric-specific scoring rules

Agent Response Format
    - JSON - recommended
    - Markdown - required fields need to be extracted
    - Text - required fields need to be extracted

Response Mapping
    - User query field
    - Reference response field
    - Candidate agent response field
    - Additional fields can be configured if required

Reference Response Type
    - HUMAN_REFERENCE
    - SME_APPROVED_AGENT_REFERENCE

The reference type should be configurable based on the available dataset.


3. Reference Response Flow
--------------------------

Scenario 1: Human Reference Response Available
----------------------------------------------

Available Data:

    User Query
        +
    Human Reference Response
        +
    Candidate Agent Response


Flow:

    User Query
        +
    Human Reference Response
        +
    Candidate Agent Response
        +
    Rubrics
        ↓
    LLM Judge
        ↓
    Correctness Score
    Completeness Score
    Reasons


Configuration:

    reference_type = HUMAN_REFERENCE


Meaning:

The human-written response acts as the trusted reference against
which the candidate agent response is evaluated.


Scenario 2: No Human Reference Response Available
-------------------------------------------------

Available Data:

    User Query
        +
    Agent V1 Response
        +
    SME Review / Score


Only Agent V1 responses that have been reviewed and approved
by the SME as correct are selected.


Flow for Creating the Reference:

    User Query
        ↓
    Agent V1 Response
        ↓
    SME Review
        ↓
    SME Confirms Response is Correct
        ↓
    Freeze Agent V1 Response
        ↓
    SME-Approved Reference Response


The same query is then executed against the candidate/current agent:


    Same User Query
        ↓
    Agent V2 / Current Agent
        ↓
    Candidate Agent Response


Evaluation Flow:

    User Query
        +
    SME-Approved Agent V1 Reference
        +
    Candidate Agent V2 Response
        +
    Rubrics
        ↓
    LLM Judge
        ↓
    Correctness Score
    Completeness Score
    Reasons


Configuration:

    reference_type = SME_APPROVED_AGENT_REFERENCE


Meaning:

There is no separately written human answer.

The SME-approved Agent V1 response becomes the trusted reference
for evaluating Agent V2 or future versions of the agent.


4. MVP Rubrics
--------------

Correctness
-----------

Checks whether the candidate agent response is factually consistent
with the trusted reference.

Example Scale:

    5 = Fully correct
    4 = Mostly correct; minor issue
    3 = Partially correct
    2 = Major factual issue
    1 = Incorrect


Completeness
------------

Checks whether the candidate agent response contains all important
information available in the trusted reference.

Example Scale:

    5 = Covers all important information
    4 = Minor omission
    3 = Important information missing
    2 = Major parts missing
    1 = Fails to cover the expected answer


5. Common Evaluation Flow
-------------------------

Regardless of the reference type, the evaluation engine receives:

    User Query
        +
    Trusted Reference Response
        +
    Candidate Agent Response
        +
    Rubrics
        +
    Scoring Rules
        ↓
    LLM Judge
        ↓
    Structured Evaluation Result


Example Output:

{
    "correctness": {
        "score": 5,
        "reason": "The candidate response is consistent with the reference."
    },
    "completeness": {
        "score": 4,
        "reason": "The answer is correct but misses one minor detail."
    }
}


6. Core Design Principle
------------------------

The evaluation engine should not care where the reference response
came from.

The configurable reference type decides how the trusted reference
is obtained:

    HUMAN_REFERENCE
        ↓
    Human-written response is used as reference


    SME_APPROVED_AGENT_REFERENCE
        ↓
    SME-approved historical agent response is used as reference


After the reference is established, the remaining evaluation flow
is identical for both scenarios.


7. High-Level Framework Flow
----------------------------

Configuration
    ↓

Load:
    - LLM Judge configuration
    - Rubrics
    - Scoring rules
    - Response format
    - Response mapping
    - Reference type
    - Retry / Timeout settings

    ↓

Prepare Evaluation Record:

    User Query
    Trusted Reference
    Candidate Agent Response

    ↓

LLM Judge

    ↓

Structured Scores + Reasons

    ↓

Store / Review Evaluation Results
