# Generation Coordinator

**Stage:** 3 of 6
**Skill:** `goga-tool-scriba-generation`
**Role:** Translation Coordinator

## Objective

Coordinate the generation of multiple translation variants by delegating to child skills.

## Manifest

```yaml
id: goga-tool-scriba-generation

consumes:
  - glossary
  - semantic_model

workflow:
  - call_skill: goga-tool-scriba-generation-literal
  - call_skill: goga-tool-scriba-generation-technical
  - call_skill: goga-tool-scriba-generation-ai

produces:
  - translation_variants
```

## Child Skills

| Skill | Variant | Goal |
|-------|---------|------|
| [generation-literal](generation-literal.md) | Literal | Maximum semantic preservation |
| [generation-technical](generation-technical.md) | Technical | Architecture-grade English |
| [generation-ai](generation-ai.md) | AI-optimized | LLM-optimized translation |

## Output

```yaml
translation_variants:
  literal: {}
  technical: {}
  ai: {}
```
