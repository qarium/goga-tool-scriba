# Context Enrichment

**Stage:** 4 of 6
**Skill:** `goga-tool-scriba-context`
**Role:** Context Enricher

## Objective

Create a unified enriched context from all prior pipeline outputs.

## Manifest

```yaml
id: goga-tool-scriba-context

consumes:
  - glossary
  - semantic_model
  - translation_variants

produces:
  - enriched_context
```

## Sources

- `glossary` — from terminology stage
- `semantic_model` — from semantic stage
- `translation_variants` — from generation stage

## Restrictions

- No new information introduced
- No structural mutations
