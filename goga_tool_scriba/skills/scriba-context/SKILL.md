---
name: goga-tool-scriba-context
description: Goga tool skill — creates a unified enriched context by merging glossary, semantic model, and translation variants without introducing new information or structural mutations. Invoked from Goga tool as part of the translation pipeline.
---

# translation-context

## Objective
Create unified enriched context.

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
- glossary
- semantic_model
- translation_variants

## Restrictions
- No new information.
- No structural mutations.
