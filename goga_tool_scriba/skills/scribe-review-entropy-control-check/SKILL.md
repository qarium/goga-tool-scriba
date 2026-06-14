---
name: goga-tool-scribe-review-entropy-control-check
description:
---

# entropy-control-check

## Manifest

```yaml
id: goga-tool-scribe-review-entropy-control-check

consumes:
  - documents

produces:
  - findings
```

## Objective
Analyze prompt documents and identify violations related to entropy control.
Focus on language that unnecessarily expands the model's solution space, increases output variance, or weakens behavioral predictability.
Detect issues that introduce excessive ambiguity, broaden generation trajectories, or reduce controllability.

Validate the following principles:
- Entropy Shaping
- Distribution Steering
- Anti-pattern Suppression

Generate findings for each detected violation.
Only report findings.

## Loop
Execute rules for 3 iterations:
1. literal analysis — evaluate only explicit document content.
2. semantic analysis — evaluate meaning using document context.
3. adversarial analysis — attempt to disprove the finding and retain only supported findings.

## Rules

### Entropy Shaping

#### Metadata

```yaml
name: entropy_shaping
category: entropy_control
```

#### Instructions
Detect optimization objectives that lack measurable success criteria.

Report a finding when ALL of the following hold:
- the instruction contains an optimization verb;
- the optimization target is not measurable;
- no acceptance criteria are provided.

Optimization verbs include:
- improve
- optimize
- enhance
- refine
- strengthen
- increase quality

Do not report findings when ANY of the following hold:
- measurable criteria are provided;
- quantitative targets are provided;
- acceptance conditions are explicitly defined;
- the optimization target is objectively verifiable.

#### Examples

##### Bad

```text
Improve the prompt.
```

##### Good

```text
Reduce prompt length by 30% without removing technical terminology.
```

### Distribution Steering

#### Metadata

```yaml
name: distribution_steering
category: entropy_control
```

#### Instructions
Detect absolute outcome requirements that are not supported by constraints or validation criteria.

Report a finding when ALL of the following hold:
- the prompt requires a guaranteed outcome;
- no supporting constraints exist;
- no examples exist;
- no acceptance criteria exist.

Examples of guaranteed outcomes:
- always generate the perfect answer;
- guarantee correctness;
- ensure the best solution;
- produce flawless output.

Do not report findings when ANY of the following hold:
- acceptance criteria are defined;
- supporting constraints are defined;
- examples are provided;
- success conditions are explicitly specified.

#### Examples

##### Bad

```text
Always generate the perfect answer.
```

##### Good

```text
Generate an answer that satisfies all listed constraints and follows the provided examples.
```

### Anti-pattern Suppression

#### Metadata

```yaml
name: anti_pattern_suppression
category: entropy_control
```

#### Instructions
Detect vague, subjective, marketing-oriented, or emotionally loaded terminology.
These terms typically increase semantic variance and reduce prompt controllability.

Report a finding when instructions rely on terms such as:
- creative
- interesting
- deep
- powerful
- innovative
- nice
- better
- amazing
- impressive
- world-class

or similar subjective descriptors.

Pay particular attention when such terms are used without objective definitions.
Require concrete and observable definitions for all subjective terms.
Do not report findings when subjective language is intentionally required by the task.

#### Examples

##### Bad

```text
Provide a powerful and innovative solution.
```

##### Good

```text
Provide a solution that minimizes latency and reduces memory consumption.
```

## Output
For every detected violation append a finding to the `pipeline_context.findings`.

**Output format:**

```yaml
findings:
  - category: entropy_control
    rule: entropy_shaping
    message: >
      Vague optimization objective detected.
    evidence: >
      Improve the prompt.
    recommendation: >
      Replace the objective with a measurable and verifiable outcome.
```