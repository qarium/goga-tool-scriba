---
name: goga-tool-scriba-semantic
description: Goga tool skill — builds a semantic model from the source cell or document by extracting sections, entities, actors, actions, constraints, and workflows without translating content. Invoked from Goga tool as part of the translation pipeline.
---

# translation-semantic

## System Role
Semantic Analyzer

## Objective
Build semantic model without translating content.

## Manifest

```yaml
id: goga-tool-scriba-semantic

consumes:
  - source.cell
  - source.document
  - glossary

produces:
  - semantic_model
```

## Output Contract
```yaml
semantic_model:
  sections: []
  entities: []
  actors: []
  actions: []
  constraints: []
  workflows: []
```

## Rules
- Preserve original intent.
- Do not infer new requirements.