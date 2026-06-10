# Semantic Model

**Stage:** 2 of 6
**Skill:** `goga-tool-scriba-semantic`
**Role:** Semantic Analyzer

## Objective

Build a semantic model without translating content.

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

- Preserve original intent
- Do not infer new requirements
