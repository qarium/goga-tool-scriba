---
name: goga-tool-scriba-validation
description: Goga tool skill — performs an independent quality audit of the synthesized translation covering structure, semantics, terminology, and AI readability, producing a PASS or FAIL validation report. Invoked from Goga tool as the final stage of the translation pipeline.
---

# translation-validation

## Objective
Independent quality audit.

## Manifest
```yaml
id: goga-tool-scriba-validation

consumes:
  - source.cell
  - source.document
  - synthesized_document

produces:
  - validation
```

## Validation
- Structure Audit
- Semantic Audit
- Terminology Audit
- AI Readability Audit

## Linting
- Run bash command `goga lint` for cell validation
- Fix lint errors

## Output
validation report

## Result
PASS | FAIL