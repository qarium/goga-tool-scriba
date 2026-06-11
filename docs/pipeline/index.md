# Translation

The translation pipeline converts technical texts — cells (CodeManifests, usage files) and standalone documents — from one language to another while preserving structure, semantics, and requirements.

The pipeline detects the source language automatically. You provide the target cell or document path and the target language.

## How it works

The pipeline runs eight stages in sequence. Each stage builds on the results of the previous one.

### 1. Terminology

Extracts technical terms, entities, and aliases from the source. Builds a canonical glossary that all subsequent stages use for consistency.

### 2. Semantic Model

Analyzes the source structure — sections, entities, actors, actions, constraints, and workflows — without translating. This model guides the generation stage.

### 3. Generation

Produces three independent translation variants:

- **Literal** — closest to the original, maximum semantic preservation
- **Technical** — architecture-grade language with active voice and industry terminology
- **AI-optimized** — explicit subjects, entities, and actions for LLM consumers

### 4. Context Enrichment

Merges the glossary, semantic model, and all three variants into a unified context — no new information is added.

### 5. Synthesis

Selects the best variant for each segment using a priority matrix: semantics first, then terminology accuracy, then readability, then AI clarity.

### 6. Validation

Runs an independent quality audit covering structure integrity, semantic fidelity, terminology consistency, AI readability, and formatting integrity. The pipeline retries on failure up to 2 times.

### 7. Apply Results

Applies synthesized documents to files at the paths defined in the cell manifest.

### 8. Finalize

Runs linting across all cells, fixes errors, and produces a human-readable report summarizing the entire pipeline output — languages, glossary, semantic model, validation results, translated files, and final status.

## What stays unchanged

Throughout all stages, the pipeline guarantees:

- Document structure
- Section and algorithm order
- Requirements and constraints
- Instruction priority

For cells, only annotations and usage files are translated — the contract structure remains untouched.
