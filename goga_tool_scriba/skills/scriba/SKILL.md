---
name: goga-tool-scriba
description: Goga tool skill — orchestrates the complete translation pipeline for cells and documents. Invoked from Goga tool to translate CodeManifests and usage files across languages with terminology analysis, semantic modeling, multi-variant generation, context enrichment, synthesis, and validation.
---

# translation

## System Role
Translation Pipeline Orchestrator

## Inputs

Detect automatically:
- Language of source. Save to `$SOURCE_LANG`.

Ask user:
- Cell or document path for translation. Save to `$DOCUMENT_PATH` or `$CELL_PATH` by type.
- Translation language. Save to `$TARGET_LANG`.

For the cell, translate the files only:
- `CODEMANIFEST` (Annotations and inline usages only)
- `<cell_name>/.usages/*.md`

## Objective
Execute the complete translation workflow using a shared `pipeline_context`.

## Execution Contract
```yaml
execution_policy:
  stop_on_error: true
  shared_context_required: true

pipeline_context:
    source:
        cell: $CELL_PATH
        document: $DOCUMENT_PATH
        language:
            source: $SOURCE_LANG
            target: $TARGET_LANG

    glossary: {}
    semantic_model: {}

    translation_variants:
        literal: {}
        technical: {}
        ai: {}

    enriched_context: {}
    synthesized_document: {}
    validation: {}
```

## Translation invariants
```yaml
translation_invariants:
  preserve_document_structure: true
  preserve_section_order: true
  preserve_list_order: true
  preserve_algorithm_order: true
  preserve_requirements: true
  preserve_constraints: true
  preserve_prompt_logic: true
  preserve_instruction_priority: true

forbidden_operations:
  - reorder_sections
  - reorder_steps
  - merge_steps
  - split_steps
  - remove_requirements
  - add_requirements
  - infer_new_logic
```

## Workflow
1. goga-tool-scriba-terminology
2. goga-tool-scriba-semantic
3. goga-tool-scriba-generation
4. goga-tool-scriba-context
5. goga-tool-scriba-synthesis
6. goga-tool-scriba-validation

## Skill Logic

Execute skills sequentially and update pipeline_context after each successful execution.

## Execution Gates
- Terminology Gate
- Semantic Gate
- Translation Gate
- Context Gate
- Synthesis Gate
- Validation Gate

## Failure Policy
stop_on_error: true
max_retries: 2

## Output
final_translation
validation_report