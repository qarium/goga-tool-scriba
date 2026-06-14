---
name: goga-tool-scribe-review-conflict-resolution
description:
---

# conflict-resolution

## Manifest

```yaml
id: goga-tool-scribe-review-conflict-resolution

consumes:
  - findings

produces:
  - findings
```

## Objective
Resolve conflicts between findings produced by validation skills.
Its responsibility is limited to determining which finding takes precedence when multiple findings target the same document fragment and propose incompatible actions.

## Conflict Definition
A conflict exists when ALL of the following hold:
- two or more findings reference the same document fragment;
- the findings belong to different categories;
- applying one recommendation would invalidate, weaken, or contradict another recommendation.

If any condition is unsatisfied, the findings are independent — preserve them as-is when:
- recommendations are compatible,
- can be applied independently,
- reference different fragments or describe different problems within the same fragment.

Multiple findings may coexist.
Retain all findings by default; suppress only findings directly involved in a resolved conflict.

## Resolution Rules

### Rule: precedence_order

Use the following category precedence order:

```yaml
priority:
  P0: blocker
  P1: critical
  P2: major
  P3: moderate
  P4: minor
  P5: trivial

  default: moderate # applied when a finding does not match any listed category

semantic_integrity: P0
prompt_framing: P1
constraint_engineering: P2
entropy_control: P3
structure_syntax: P4
efficiency: P5
```

When two or more findings conflict:
- the finding with the highest priority (lowest P-number) becomes the primary finding;
- the lower priority finding becomes a secondary finding — retained but marked as overridden.

### Rule: equal_precedence
When conflicting findings belong to the same category:
- select the finding that references exact line numbers and quotes the source text;
- if both findings are equally specific, preserve both findings.

## Workflow
1. Read all findings.
2. Group findings by referenced document fragment.
3. Detect conflicts.
4. Apply category precedence.
5. Mark primary findings.
6. Mark secondary findings.
7. Return updated findings.

## Output
Append the following fields when a conflict is resolved.

```yaml
findings:
  - category: <category>
    rule: <rule>
    message: >
      <message>
    evidence: >
      <evidence>
    recommendation: >
      <recommendation>
```