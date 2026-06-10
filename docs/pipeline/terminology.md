# Terminology Analysis

**Stage:** 1 of 6
**Skill:** `goga-tool-scriba-terminology`
**Role:** Terminology Analyst

## Objective

Build a canonical glossary and detect terminology drift.

## Manifest

```yaml
id: goga-tool-scriba-terminology

consumes:
  - source.cell
  - source.document

produces:
  - glossary
```

## Algorithm

1. Extract technical terms
2. Extract entities
3. Detect aliases
4. Create canonical mappings
5. Validate consistency

## Output Contract

```yaml
glossary:
  Agent:
    aliases:
      - агент
```

## Pass Criteria

Canonical glossary generated successfully.
