# Validation

**Stage:** 6 of 6
**Skill:** `goga-tool-scriba-validation`
**Role:** Validator

## Objective

Independent quality audit of the synthesized translation.

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

## Audit Areas

1. **Structure Audit** — document structure integrity
2. **Semantic Audit** — semantic fidelity to source
3. **Terminology Audit** — glossary consistency
4. **AI Readability Audit** — clarity for AI consumers

## Linting

The validation stage runs linting via:

```bash
goga lint
```

This validates all cells without arguments. Lint errors must be fixed before the validation can pass.

## Output

Validation report with verdict: **PASS** or **FAIL**.
