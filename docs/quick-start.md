# Quick Start

## 1. Install the tool

```bash
pip install goga-tool-scriba
```

## 2. Connect to an agent

```bash
goga connect <agent>
```

## 3. Invoke the dispatcher

The main entry point is the command `/goga:tool scriba`. Run it with a request describing what you want; the dispatcher detects the operation and routes to the matching pipeline.

### Examples

```
/goga:tool scriba translate cell path/to/cell to French
/goga:tool scriba переведи cell path/to/cell на английский
/goga:tool scriba review cell path/to/cell
/goga:tool scriba проверить cell path/to/cell
/goga:tool scriba validate prompts in cell path/to/cell
```

### Intent detection

- Use words like "translate" / "перевести" and mention a target language to run the **translation** pipeline.
- Use words like "review" / "проверить" / "prompt engineering check" / "validate prompts" / "find issues" to run the **review** pipeline.
- If the intent is ambiguous, the dispatcher asks you to confirm.

## Translation pipeline

For translation, the dispatcher routes to `goga-tool-scriba-trans`, which:

1. **Detects** the source language automatically
2. **Asks** for the target cell/document path and target language
3. **Executes** the 8-stage translation pipeline
4. **Outputs** the final translation with a quality audit report

### Example

```
/goga:tool scriba translate cell path/to/cell to English
```

### Pipeline stages

| Stage | Skill | Purpose |
|-------|-------|---------|
| 1 | Terminology | Build canonical glossary |
| 2 | Semantic | Build semantic model |
| 3 | Generation | Produce literal, technical, and AI variants |
| 4 | Context | Merge into unified enriched context |
| 5 | Synthesis | Select best variant per segment |
| 6 | Validation | Quality audit (PASS/FAIL) |
| 7 | Apply results | Write synthesized documents to manifest paths |
| 8 | Finalize | Lint, fix, and produce final report |

See [Translation pipeline](pipeline/index.md) for details.

## Review pipeline

For prompt-engineering review, the dispatcher routes to `goga-tool-scribe-review`, which:

1. **Asks** for the target cell/document path
2. **Executes** the 10-stage review pipeline
3. **Outputs** a findings collection, applied fixes, and a final validation status

### Example

```
/goga:tool scriba review cell path/to/cell
```

See [Review pipeline](review/index.md) for details.

## What gets processed

For cells, only these files are affected by either pipeline:

- `CODEMANIFEST` — annotations and inline usages only
- `<cell_name>/.usages/*.md`

For standalone documents, the full content is processed.

## Invariants

Both pipelines guarantee these invariants throughout processing:

- Document structure preserved
- Section order preserved
- List order preserved
- Algorithm order preserved
- Requirements preserved
- Constraints preserved
- Prompt logic preserved
- Instruction priority preserved
