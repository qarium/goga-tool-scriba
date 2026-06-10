# Synthesis

**Stage:** 5 of 6
**Skill:** `goga-tool-scriba-synthesis`
**Role:** Synthesizer

## Objective

Produce the final translation by selecting the best variant per segment.

## Manifest

```yaml
id: goga-tool-scriba-synthesis

consumes:
  - glossary
  - semantic_model
  - translation_variants
  - enriched_context

produces:
  - synthesized_document
```

## Decision Matrix

The synthesis selects the best variant for each segment based on the following priorities:

| Priority | Dimension | Source |
|----------|-----------|--------|
| 1 | Semantics | Literal variant |
| 2 | Terminology | Glossary |
| 3 | Readability | Technical variant |
| 4 | AI Clarity | AI variant |

## Mandatory Invariants

- Preserve document structure
- Preserve algorithm order
- Preserve instruction priority

## Output

`synthesized_document`
