# goga-tool-scriba

Goga tool for working with technical texts — processing cells (CodeManifests, usage files) and standalone documents while preserving structure, semantics, and requirements.

**Documentation:** [qarium.github.io/goga-tool-scriba](https://qarium.github.io/goga-tool-scriba/)

## Installation

```bash
pip install goga-tool-scriba
```

## Connecting to an agent

```bash
goga connect <agent>
```

## Features

### Translation

Invoked via the `goga-tool-scriba` skill, which orchestrates a sequential pipeline:
- terminology analysis,
- semantic modeling,
- multi-variant generation,
- context enrichment,
- synthesis,
- and validation — producing a final translation with a quality audit report.

The pipeline automatically detects the source language and asks for the target document/cell path and target language.
It preserves document structure, section order, algorithm order, requirements, constraints, and instruction priority throughout the process.
Translating a cell affects only CodeManifest annotations, inline usages, and `.usages/*.md` files.

The pipeline stops on the first error and retries up to 2 times before failing.