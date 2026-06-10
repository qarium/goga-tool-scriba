# Quick Start

## 1. Install the tool

```bash
pip install goga-tool-scriba
```

## 2. Connect to an agent

```bash
goga connect <agent>
```

## 3. Invoke the translation skill

Trigger the `goga-tool-scriba` skill in your agent session. The pipeline will:

1. **Detect** the source language automatically
2. **Ask** for the target cell/document path and target language
3. **Execute** the 6-stage translation pipeline
4. **Output** the final translation with a quality audit report

## Pipeline stages

| Stage | Skill | Purpose |
|-------|-------|---------|
| 1 | Terminology | Build canonical glossary |
| 2 | Semantic | Build semantic model |
| 3 | Generation | Produce literal, technical, and AI variants |
| 4 | Context | Merge into unified enriched context |
| 5 | Synthesis | Select best variant per segment |
| 6 | Validation | Quality audit (PASS/FAIL) |

## What gets translated

For cells, only these files are affected:

- `CODEMANIFEST` — annotations and inline usages only
- `<cell_name>/.usages/*.md`

For standalone documents, the full content is translated.

## Invariants

The pipeline guarantees these invariants throughout translation:

- Document structure preserved
- Section order preserved
- Algorithm order preserved
- Requirements preserved
- Constraints preserved
- Instruction priority preserved
