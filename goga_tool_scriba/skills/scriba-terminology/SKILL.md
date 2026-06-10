---
name: goga-tool-scriba-terminology
description: Goga tool skill — builds a canonical glossary by extracting technical terms, entities, and aliases, then creating canonical mappings and validating consistency. Invoked from Goga tool as the first stage of the translation pipeline.
---

# translation-terminology

## System Role
Terminology Analyst

## Objective
Build canonical glossary and detect terminology drift.

## Manifest

```yaml
id: goga-tool-scriba-terminology

consumes:
  - source.documents

produces:
  - glossary
```

## Algorithm
1. Extract technical terms.
2. Extract entities.
3. Detect aliases.
4. Create canonical mappings.
5. Validate consistency.

## Output Contract

```yaml
glossary:
  Agent:
    aliases:
      - агент
```

## PASS
Canonical glossary generated.