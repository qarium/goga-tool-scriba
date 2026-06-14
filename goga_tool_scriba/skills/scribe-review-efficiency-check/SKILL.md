---
name: goga-tool-scribe-review-efficiency-check
description:
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
Detect language that increases prompt size without adding meaningful information.

Every token must contribute to:
- intent definition;
- constraint definition;
- task execution;
- output specification;
- context required for successful completion.

Report a finding when:
- filler language dominates an instruction;
- multiple phrases communicate the same requirement;
- excessive politeness obscures intent;
- motivational language replaces actionable guidance;
- verbosity reduces information density;
- instructions contain unnecessary repetition.

Pay particular attention to phrases such as:
- please try your best
- carefully analyze
- pay close attention
- make sure to
- I would like you to
- it is very important that
- do your best
- take your time
- think carefully

Report findings only when the phrase does not add operational meaning.

Do not report findings when repetition is intentionally used to reinforce critical constraints.

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