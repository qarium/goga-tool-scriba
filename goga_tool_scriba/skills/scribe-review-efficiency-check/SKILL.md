---
name: goga-tool-scribe-review-efficiency-check
description: Goga tool skill — review stage that validates prompt efficiency. Detects token economy violations — filler language, motivational phrasing, and redundant repetitions that increase token count without adding operational meaning. Consumes documents, produces findings.
---

# efficiency-check

## Manifest

```yaml
id: goga-tool-scribe-efficiency-check

consumes:
  - documents

produces:
  - findings
```

## Objective
Analyze prompt documents and identify violations related to prompt efficiency.
Focus on information density, redundancy, and unnecessary token consumption.
Detect issues that increase prompt size without increasing instruction quality, constraint clarity, or execution reliability.

Validate the following principle:
- Token Economy

Generate findings for each detected violation.
Only report findings.

## Loop
Execute rules for 3 iterations:
1. literal analysis — evaluate only explicit document content.
2. semantic analysis — evaluate meaning using document context.
3. adversarial analysis — attempt to disprove the finding and retain only supported findings.

## Rules

### Token Economy

#### Metadata

```yaml
name: token_economy
category: efficiency
```

#### Instructions
Your task is to detect prompt bloat — the presence of filler language, excessive politeness, motivational phrasing, or redundant repetitions that increase token count without adding operational meaning or constraints.

To ensure 100% deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Filler Phrase & Bloat Token Extraction
Scan the entire input text and identify all sentences or phrases that fall into the following linguistic bloat categories (case-insensitive):
1. [POLITENESS_FILLER]: "please", "i would like you to", "could you please", "kindly", "if you don't mind".
2. [EMOTIONAL_MOTIVATIONAL]: "try your best", "do your best", "take your time", "think carefully", "pay close attention", "carefully analyze", "it is very important that", "make sure to".
3. [REDUNDANT_DUPLICATION]: The exact same constraint or task requirement is stated 2 or more times using different words within adjacent sentences, without introducing any new metrics or parameters.

Record the total count of these detections under [TOTAL_BLOAT_COUNT]. If [TOTAL_BLOAT_COUNT] == 0, immediately terminate and output OK.

**STEP 2:** Operational Meaning & Reinforcement Audit
For each flagged element from Step 1, verify if it serves a legitimate engineering purpose. Set IS_FUNCTIONAL = TRUE if and only if the phrase or block satisfies at least one of the following conditions:
1. [CRITICAL_REINFORCEMENT]: The repetition is intentionally used to reinforce a high-stakes constraint (e.g., safety, data privacy, or a formatting rule that was separated by a large block of text).
2. [EXAMPLE_QUOTATION]: The filler phrase appears strictly inside text explicitly designated as an Example or Quote (e.g., "The user might say: 'Please help me'").

If a filler phrase is used simply as a prefix to a core task or command (e.g., "I would like you to carefully analyze the document"), set IS_FUNCTIONAL = FALSE.

**STEP 3:** Strict Score Evaluation
Evaluate the extracted markers using the following strict logical matrix:
- Rule 1 (Information Density Deficit): [TOTAL_BLOAT_COUNT] >= 1, AND at least one flagged phrase has the status IS_FUNCTIONAL == FALSE. (This is a direct violation: empty tokens obscure the execution path).
- Rule 2 (Compliant Content): All phrases in the text are functional, or no filler tokens are detected.

#### Examples

##### Bad

```text
I would like you to carefully analyze the document and do your best to provide the most accurate answer possible.
```

##### Good

```text
Analyze the document and provide an accurate answer.
```

## Output
For every detected violation append a finding to the `pipeline_context.findings`.

**Output format:**

```yaml
findings:
  - category: efficiency
    rule: token_economy
    message: >
      Unnecessary prompt verbosity detected.
    evidence: >
      I would like you to carefully analyze the document and do your best.
    recommendation: >
      Remove filler language and express the requirement directly.
```