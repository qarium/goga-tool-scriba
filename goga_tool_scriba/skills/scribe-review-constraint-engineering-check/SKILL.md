---
name: goga-tool-scribe-review-constraint-engineering-check
description:
---

# constraint-engineering-check

## Manifest

```yaml
id: goga-tool-scribe-review-constraint-engineering-check

consumes:
  - documents

produces:
  - findings
```

## Objective
Analyze prompt documents and identify violations related to constraint engineering.
Focus on how requirements, limitations, and behavioral constraints are defined, prioritized, and reinforced.
Detect issues that reduce constraint compliance, create conflicting requirements, produce non-deterministic output for identical inputs, or make instructions ambiguous to the executing agent.

Validate the following principles:
- Token Inertia
- Negative Instruction
- Constraint Compatibility
- Lexical Determinism

Generate findings for each detected violation.
Report findings without altering original prompt content.

## Loop
Execute rules for 3 iterations:
1. literal analysis — evaluate only explicit document content.
2. semantic analysis — evaluate meaning using document context.
3. adversarial analysis — attempt to disprove the finding and retain only supported findings.

## Rules

### Token Inertia

#### Metadata

```yaml
name: token_inertia
category: constraint_engineering
```

#### Instructions
Detect output-format, compliance, and safety constraints positioned too far from the point of execution.
Language models tend to give greater weight to recent instructions than distant instructions.

Report a finding when:
- critical output constraints appear only at the beginning of a long prompt;
- format requirements are separated from output instructions by large amounts of content;
- important restrictions are introduced early and never reinforced;
- execution requirements are likely to be weakened by prompt length.

Pay particular attention to:
- output formats;
- content restrictions (what may or may not appear in the output);
- compliance requirements (rules the output must satisfy).

Prefer placing critical constraints near the execution task or output definition.
Do not report findings when prompts are short enough that instruction decay is unlikely.

#### Examples

##### Bad

```text
Do not use markdown.

...

[many sections later]

Generate the final response.
```

##### Good

```text
Generate the final response.

Do not use markdown.
```

### Negative Instruction

#### Metadata

```yaml
name: negative_instruction
category: constraint_engineering
```

#### Instructions
Detect constraints expressed primarily through negation.
Negative instructions often activate the very concepts they attempt to suppress.

Report a finding when:
- requirements are defined as "do not";
- behavior is described through prohibited outcomes;
- restrictions rely on negated concepts rather than desired behavior;
- multiple negative instructions are chained together.

Prefer describing the desired behavior directly.
Do not report findings when negation is necessary to express a safety, compliance, or legal requirement.

#### Examples

##### Bad

```text
Do not use complex sentences.
```

##### Good

```text
Use short sentences containing 5 to 12 words.
```

### Constraint Compatibility

#### Metadata

```yaml
name: constraint_compatibility
category: constraint_engineering
```

#### Instructions
Detect constraints that cannot be satisfied simultaneously. This includes constraints that are mutually exclusive (cannot both hold), that compete for limited resources (e.g. brevity vs. completeness), or that directly contradict each other (one negates the other).
A prompt should define a coherent set of requirements.

Report a finding when:
- brevity conflicts with completeness;
- determinism conflicts with creativity;
- strict compliance conflicts with unrestricted exploration;
- multiple objectives compete for priority;
- requirements cannot be satisfied simultaneously.

Look for instructions that force trade-offs without establishing precedence.
Prefer explicit prioritization of constraints.
Do not report findings when priorities are clearly defined.

#### Examples

##### Bad

```text
Be brief, exhaustive, and highly detailed.
```

##### Good

```text
Priority:

1. Accuracy
2. Brevity
3. Completeness
```

### Lexical Determinism

#### Metadata

```yaml
name: lexical_determinism
category: constraint_engineering
```

#### Instructions
Detect weak, non-committal, or probabilistic instruction language.
Constraint strength is influenced by lexical choice.

Report a finding when requirements rely on terms such as:
- try
- preferably
- maybe
- could
- if possible
- ideally

These terms weaken compliance expectations and increase behavioral variance.
Prefer explicit and deterministic language.

Examples of stronger alternatives include:
- must
- exactly
- only
- strict
- required
- mandatory

Do not report findings when optional behavior is genuinely intended.

#### Examples

##### Bad

```text
Try to follow the specification.
```

##### Good

```text
Follow the specification exactly.
```

## Output
For every detected violation append a finding to the `pipeline_context.findings`.

**Output format:**

```yaml
findings:
  - category: constraint_engineering
    rule: constraint_compatibility
    message: >
      Conflicting constraints detected.
    evidence: >
      Be brief, exhaustive, and highly detailed.
    recommendation: >
      Define explicit constraint priorities and remove incompatible requirements.
```