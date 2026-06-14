---
name: goga-tool-scribe-review-prompt-framing-check
description:
---

# prompt-framing-check

## Manifest

```yaml
id: goga-tool-scribe-review-prompt-framing-check

consumes:
  - documents

produces:
  - findings
```

## Objective
Analyze prompt documents and identify violations related to prompt framing.
Focus on how the prompt establishes, maintains, and reinforces the intended generation mode.
Detect issues that cause frame drift, latent state contamination, incorrect inference mode activation, or weak behavioral anchoring.

Validate the following principles:
- Embedding Alignment
- Frame Locking
- Semantic Priming
- Latent State Persistence

Generate findings for each detected violation.
Only report findings.

## Loop
Execute very rules for 3 iterations:
1. local analysis — analyze the fragment in isolation.
2. context analysis — analyze the fragment in the context of the entire document.
3. conflict analysis — verify that the finding remains valid when considering all surrounding instructions.

## Rules

### Embedding Alignment

#### Metadata

```yaml
name: embedding_alignment
category: prompt_framing
```

#### Instructions
Detect language that activates behavioral modes inconsistent with the prompt objective.
Words and phrases influence latent-space activation and inference behavior.

Report a finding when:
- creative terminology appears in deterministic workflows;
- brainstorming language appears in validation workflows;
- exploratory language appears in specification workflows;
- divergent and constrained generation signals are mixed.

Pay particular attention to terms such as:
- creative
- brainstorm
- imagine
- interesting
- innovative
- explore
- think freely

Use terminology aligned with the intended execution mode.

Examples of deterministic terminology include:
- strict
- exact
- deterministic
- formal specification
- validate
- verify
- compliant

Do not report findings when exploratory behavior is the explicit objective.

#### Examples

##### Bad

```text
Generate a strict technical specification.
Use creative and innovative approaches.
```

##### Good

```text
Generate a strict technical specification.
Use deterministic reasoning and exact terminology.
```

### Frame Locking

#### Metadata

```yaml
name: frame_locking
category: prompt_framing
```

#### Instructions
Detect incompatible execution frames within the same execution scope.

Report a finding when ALL of the following hold:
- two or more execution frames are present;
- at least one pair of frames belongs to incompatible categories;
- the prompt does not explicitly separate them into distinct phases.

Treat the following frame pairs as incompatible:
- analytical ↔ entertaining
- specification ↔ storytelling
- deterministic ↔ brainstorming
- validation ↔ roleplay
- compliance ↔ creative exploration

Do not report findings when ANY of the following hold:
- the frames are executed in separate phases;
- an explicit transition exists between the frames;
- one frame is provided only as an example;
- the document clearly establishes phase boundaries.

#### Examples

##### Bad

```text
Analyze the architecture in a fun and emotional way.
```

##### Good

```text
Analyze the architecture using a formal engineering approach.
```

### Semantic Priming

#### Metadata

```yaml
name: semantic_priming
category: prompt_framing
```

#### Instructions
Detect delayed establishment of execution context.

Report a finding when ALL of the following hold:
- a role definition exists;
- the role definition appears after the first task statement or behavioral constraint;
- moving the role definition earlier would change interpretation of subsequent instructions.

Role definitions include:
- expertise declarations;
- persona definitions;
- authority statements;
- execution responsibilities.

Do not report findings when ANY of the following hold:
- the role definition appears before the first task;
- the document contains no role definition;
- the role is established in the document title or opening section.

#### Examples

##### Bad

```text
Let's think together.

...

You are a senior compiler engineer.
```

##### Good

```text
You are a senior compiler engineer.
Analyze the implementation according to compiler design principles.
```

### Latent State Persistence

#### Metadata

```yaml
name: latent_state_persistence
category: prompt_framing
```

#### Instructions
Detect transitions between incompatible execution modes without explicit re-grounding.

Report a finding when ALL of the following hold:
- execution mode A is established;
- execution mode B is established later;
- mode A and mode B belong to different behavioral categories;
- no explicit transition or reset instruction exists.

Examples of explicit transitions:
- End roleplay.
- Switch to technical mode.
- Use a formal engineering style.
- Reset previous assumptions.

Do not report findings when ANY of the following hold:
- an explicit transition exists;
- only one execution mode is present;
- the second mode is clearly scoped as an example;
- the document explicitly resets context.

#### Examples

##### Bad

```text
Imagine you are a medieval king.

...

Analyze the API specification.
```

##### Good

```text
Imagine you are a medieval king.

...

End roleplay.
Use a formal technical style.
Analyze the API specification.
```

## Output
For every detected violation append a finding to the `pipeline_context.findings`.

**Output format:**

```yaml
findings:
  - category: prompt_framing
    rule: frame_locking
    message: >
      Conflicting execution frames detected.
    evidence: >
      Analyze the architecture in a fun and emotional way.
    recommendation: >
      Use a single execution frame and remove conflicting behavioral signals.
```