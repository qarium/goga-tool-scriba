# Translation Pipeline

The `goga-tool-scriba` skill orchestrates a complete translation pipeline for cells and standalone documents.

## Execution Model

The pipeline runs stages sequentially, updating a shared `pipeline_context` after each successful execution.

```yaml
execution_policy:
  stop_on_error: true
  max_retries: 2
  shared_context_required: true
```

## Shared Context

All stages read from and write to a shared context:

```yaml
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

## Stage Order

1. [Terminology](terminology.md) — canonical glossary extraction
2. [Semantic Model](semantic.md) — semantic structure extraction
3. [Generation](generation.md) — multi-variant translation (literal, technical, AI)
4. [Context Enrichment](context.md) — unified context merging
5. [Synthesis](synthesis.md) — best-variant selection per segment
6. [Validation](validation.md) — independent quality audit

## Execution Gates

Each stage has an execution gate that must pass before the next stage begins:

- Terminology Gate
- Semantic Gate
- Translation Gate
- Context Gate
- Synthesis Gate
- Validation Gate

## Failure Policy

```yaml
stop_on_error: true
max_retries: 2
```

The pipeline stops on the first error and retries up to 2 times before failing.

## Translation Invariants

The pipeline enforces these invariants throughout all stages:

| Invariant | Description |
|-----------|-------------|
| `preserve_document_structure` | Document structure remains unchanged |
| `preserve_section_order` | Section ordering is maintained |
| `preserve_list_order` | List ordering is maintained |
| `preserve_algorithm_order` | Algorithm step ordering is maintained |
| `preserve_requirements` | All requirements are retained |
| `preserve_constraints` | All constraints are retained |
| `preserve_prompt_logic` | Prompt logic is unchanged |
| `preserve_instruction_priority` | Instruction priority is maintained |

### Forbidden operations

- Reorder sections
- Reorder steps
- Merge steps
- Split steps
- Remove requirements
- Add requirements
- Infer new logic

## Output

The pipeline produces:

- **`final_translation`** — the synthesized translation document
- **`validation_report`** — quality audit with PASS or FAIL verdict
