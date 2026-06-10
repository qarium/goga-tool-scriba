---
name: goga-tool-scriba-synthesis
description: Goga tool skill — produces the final synthesized translation by selecting the best variant per segment using a semantics-terminology-readability-AI clarity decision matrix. Invoked from Goga tool as part of the translation pipeline.
---

# translation-synthesis

## Objective
Produce final translation.

## Manifest
```yaml
id: goga-tool-scriba-synthesis

consumes:
  - glossary
  - semantic_model
  - translation_variants
  - enriched_context

produces:
  - synthesized_documents
```

## Decision Matrix
Semantics -> Literal
Terminology -> Glossary
Readability -> Technical
AI Clarity -> AI Variant

## Mandatory Invariants
- Preserve document structure.
- Preserve algorithm order.
- Preserve instruction priority.

## Output
synthesized_documents