---
name: goga-tool-scriba-generation
description: Goga tool skill — coordinates the generation of multiple translation variants (literal, technical, AI-optimized) by delegating to child skills. Invoked from Goga tool as part of the translation pipeline.
---

# translation-generation

## System Role
Translation Coordinator

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
- goga-tool-scriba-generation-literal
- goga-tool-scriba-generation-technical
- goga-tool-scriba-generation-ai

## Output
translation_variants