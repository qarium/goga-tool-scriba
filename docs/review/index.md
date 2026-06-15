# Review pipeline

The review pipeline validates prompt documents — cells (CodeManifests, usage files) and standalone documents — against established prompt-engineering principles.

## How to invoke

Run `/goga:tool scriba` with a request that signals review intent. The dispatcher routes to the `goga-tool-scribe-review` skill, which runs the pipeline.

### Examples

```
/goga:tool scriba review cell path/to/cell
/goga:tool scriba проверить cell path/to/cell
/goga:tool scriba validate prompts in cell path/to/cell
/goga:tool scriba find prompt issues in documents path/to/docs
```

You provide the target cell or document path. The pipeline collects findings, resolves conflicts, plans and applies fixes, and validates the result.

## How it works

The pipeline runs ten stages in sequence. Each stage writes its output into a shared `pipeline_context`.

### Checks

Six independent category skills run first, each producing findings in its own domain.

#### 1. Structure & syntax

Validates document structure, sectioning, list and step ordering, and syntactic well-formedness.

#### 2. Prompt framing

Checks how instructions are framed — anchors, modality, scope, and the clarity of the agent's role and objective.

#### 3. Constraint engineering

Audits constraints for soundness: unambiguous modal verbs, absence of contradictions, and feasibility of enforcement.

#### 4. Entropy control

Detects under- and over-constrained fragments — places where the model has too much freedom or too little, increasing the risk of drift.

#### 5. Semantic integrity

Ensures the semantic intent is preserved across sections, aliases, and references — no implicit redefinition of terms or scope.

#### 6. Efficiency

Identifies redundancy, verbosity, and low-signal instructions that increase prompt cost without improving outcomes.

### Resolution and fixes

#### 7. Conflict resolution

Resolves contradictions between findings produced by different category skills. Preserves findings exactly as generated and produces a single, non-conflicting collection.

#### 8. Planner

Builds a structured fix plan from the resolved findings — one entry per actionable change.

#### 9. Fix

Applies the plan to the documents, modifying annotations and usage files only.

#### 10. Validation

Re-checks the patched documents against the same principles and produces a PASS/FAIL status. The pipeline retries on failure up to 2 times.

## Strict preservation rules

For cells, the review operates under strict preservation constraints.

### CodeManifest annotations

- **Strict preservation** — all usage links enclosed in backticks (e.g., `usage`) must be retained; modifying or removing the backticks or the enclosed text is strictly prohibited.
- **Scope** — annotations are exclusively high-level requirements and algorithmic logic; technical implementation details are out of scope.
- **Refinement rule** — rephrasing during review is permitted, provided the semantic intent stays strictly within requirements and algorithms.

### Usage files

- **Strict preservation** — all code examples must be retained verbatim; modifying, truncating, or omitting code blocks is strictly prohibited.
- **Scope** — usage files are exclusively practical usage examples; implementation requirements are out of scope.
- **Refinement rule** — rephrasing during review is permitted, provided the semantic intent stays strictly confined to demonstrating usage.

## What stays unchanged

Throughout all stages, the review pipeline guarantees:

- Document structure
- Section and list order
- Algorithm order
- Requirements and constraints
- Prompt logic
- Instruction priority

Forbidden operations: reordering sections or steps, merging or splitting steps, removing or adding requirements, inferring new logic.

## See also

- [Translation pipeline](../pipeline/index.md) — the multi-language translation workflow.
- [Quick Start](../quick-start.md) — installation and a brief walkthrough of both pipelines.
