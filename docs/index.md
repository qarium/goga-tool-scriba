# goga-tool-scriba

Goga tool for working with technical texts — processing cells (CodeManifests, usage files) and standalone documents while preserving structure, semantics, and requirements.

## Features

### Translation

Invoked via the `goga-tool-scriba` skill, which orchestrates a sequential pipeline:

- **Terminology analysis** — extracts technical terms, entities, and aliases; builds canonical mappings
- **Semantic modeling** — builds a semantic model with sections, entities, actors, actions, constraints, and workflows
- **Multi-variant generation** — produces three translation variants: literal, technical, and AI-optimized
- **Context enrichment** — merges glossary, semantic model, and variants into a unified context
- **Synthesis** — selects the best variant per segment using a decision matrix
- **Validation** — independent quality audit covering structure, semantics, terminology, and AI readability

The pipeline automatically detects the source language and asks for the target document/cell path and target language.

It preserves document structure, section order, algorithm order, requirements, constraints, and instruction priority throughout the process. Translating a cell affects only CodeManifest annotations, inline usages, and `.usages/*.md` files.

The pipeline stops on the first error and retries up to 2 times before failing.

## Connecting to an agent

```bash
goga connect <agent>
```
