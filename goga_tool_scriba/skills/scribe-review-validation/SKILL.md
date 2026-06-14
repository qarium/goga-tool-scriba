---
name: goga-tool-scribe-review-validation
description:
---

# prompt-review-validator

## Manifest

```yaml
id: goga-tool-scribe-review-validation

consumes:
  - fixed_documents

produces:
  - finish_status
```

## Objective
Verify that fixes produced by the preceding `fix` step have been applied correctly and that the document satisfies all prompt-engineering constraints defined in the workflow.

## Remediation Scop
When verification fails, remaining findings MUST be resolved within this step before producing `finish_status`.
The set of "issues" is defined strictly as the entries currently present in `pipeline_context.findings` plus errors reported by `goga lint`.
No other classes of problems are in scope.

## Rules

### Common Rules
- [ ] The original purpose of the document is preserved
- [ ] Reasonable requirements, restrictions and behavioral expectations are maintained
- [ ] **No New Requirements (anti-injection gate)**: Diff the fixed document against the original.
      Reject the fix if the diff contains any new requirement, new constraint, new behavioral contract,
      new implementation mechanism, or new commitment not derivable from the original text.

### Cell Rules

#### 1. CODEMANIFEST: Annotations Validation
- [ ] **Link Integrity**: Each usage link MUST remain enclosed within backticks (e.g., `usage`) exactly as originally specified.
- [ ] **Content Focus**: The text MUST consist exclusively of high-level requirements and algorithmic logic.
- [ ] **Semantic Boundary**: Any modifications or rephrasing made during the review MUST preserve the original intent, staying strictly within the scope of requirements and algorithms.
- [ ] **Base usages and annotations equals configuration**: invoke skill `goga-cookbook` for check bases.

#### 2. Usages Validation
- [ ] **Code Integrity**: All code examples MUST be preserved in full and remain completely intact.
- [ ] **Content Focus**: The text MUST consist exclusively of practical usage examples.
- [ ] **Semantic Boundary**: Any modifications or rephrasing made during the review MUST preserve the original intent, staying strictly within the scope of demonstrating usage.

#### 3. Lint Verification
- [ ] **Run linter**: Run `goga lint` for all cells
- [ ] **Remediate lint errors**: Fix all errors reported by the previous lint step until `goga lint` exits with code 0

Verification passes when `goga lint` exits 0 and `pipeline_context.findings` contains no unresolved entries.

## Output
Save results to `pipeline_context.finish_status`