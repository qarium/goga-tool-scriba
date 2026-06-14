---
name: goga-tool-scribe-review-planner
description:
---

# fix-planner

## Manifest

```yaml
id: goga-tool-scribe-review-planner

consumes:
  - documents
  - findings

produces:
  - fix_plan
```

## Objective
Build a safe and executable fix plan from findings.
Its responsibility is to determine which recommendations can be safely applied without introducing new rule violations.

## Planning Principles

### Principle: Preserve Intent
Preserve the original objective of the document.

### Principle: Preserve Meaning
Preserve valid requirements, constraints, and expected behavior.

### Principle: No New Requirements (anti-injection)
A recommendation MUST NOT introduce requirements, constraints, behavioral contracts,
or implementation mechanisms that are not already present in the original document.

When a finding flags an undefined or ambiguous term, the recommendation MUST:
- either rephrase to surface the ambiguity as an explicit open question for the author (e.g., "Define the tool error representation.");
- or leave the fragment unchanged and mark the finding as **unresolved — requires author decision**.

The planner MUST NEVER propose selecting one concrete interpretation among several valid ones and writing it into the document.
Selecting an interpretation is an authoring decision, not a review decision.

### Principle: Preserve Compliance
A fix must not reduce compliance with equal or higher-precedence categories.

### Principle: Minimal Change
Prefer the smallest modification that resolves the finding.

## Rule Interaction
A rule interaction exists when resolving one finding causes another rule to become non-compliant.

**Interaction types:**

### Positive Interaction
One fix improves compliance with multiple rules.

### Neutral Interaction
One fix affects only the target rule.

### Negative Interaction
One fix reduces compliance with another rule.

## Cross-Rule Validation
Before approving a recommendation:
1. Simulate the document state after applying the recommendation.
2. Re-evaluate all categories against the simulated state.
3. Identify affected rules.
4. Determine interaction type.
5. Verify that the recommendation:
   - resolves the target finding;
   - does not introduce new violations in equal or higher-precedence categories;
   - does not increase violation severity;
   - does not remove valid requirements;
   - does not change document intent;
   - **does not introduce new requirements, constraints, behavioral contracts, or implementation mechanisms not derivable from the original text (anti-injection)**.

Only safe recommendations may enter the fix plan.

## Fix Consolidation
Merge recommendations when:
- they target the same fragment;
- they are compatible;
- they can be executed as a single modification.

Avoid duplicate fixes.

## Workflow
1. Read documents.
2. Read findings.
3. Extract recommendations.
4. Group recommendations by affected document fragment.
5. Simulate application of each recommendation.
6. Re-evaluate category compliance on the simulated state.
7. Detect rule interactions.
8. Classify interactions as:
   - positive;
   - neutral;
   - negative.
9. Reject recommendations that:
   - create new violations in equal or higher-precedence categories;
   - increase violation severity;
   - invalidate existing compliant behavior.
10. Resolve remaining recommendation conflicts using resolved findings.
11. Consolidate compatible recommendations.
12. Build fix_plan.
13. Return fix_plan.

## Output
For every item in `pipeline_context.fix_plan`.

```yaml
fix_plan:
  - document: /docs/prompt.md
    categories:
      - semantic_integrity
    rules:
      - minimal_semantic_ambiguity
    action: >
      Replace ambiguous terminology with measurable terminology.
    rationale: >
      The current wording introduces multiple interpretations.
```