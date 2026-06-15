# goga-tool-scriba

Goga tool for working with technical texts — processing cells (CodeManifests, usage files) and standalone documents while preserving structure, semantics, and requirements.

## How to invoke

The main entry point and the top-level orchestrator is the command:

```
/goga:tool scriba <request>
```

When you run `/goga:tool scriba`, it activates the `goga-tool-scriba` skill — a dispatcher that detects operation intent from your request and routes to the matching pipeline. You do not invoke the underlying pipeline skills directly; the dispatcher does it for you.

### Examples

```
/goga:tool scriba translate cell path/to/cell to French
/goga:tool scriba переведи cell path/to/cell на английский
/goga:tool scriba review cell path/to/cell
/goga:tool scriba проверить cell path/to/cell
/goga:tool scriba validate prompts in cell path/to/cell
```

## Dispatcher

The `goga-tool-scriba` skill is a top-level dispatcher that detects operation intent from the request and routes to the matching pipeline:

| Signal in request | Operation | Routed skill |
|---|---|---|
| "translate", "перевести", target language, source/target language pair | translation | `goga-tool-scriba-trans` |
| "review", "проверить", "ревью", "prompt engineering check", "validate prompts", "find issues" | review | `goga-tool-scribe-review` |
| Both intents, or ambiguous / unstated | ask | dispatcher asks the user to confirm |

The dispatcher owns no inputs or pipeline state — the dispatched skill handles its own onboarding.

## Features

### Translation

Translation pipeline (`goga-tool-scriba-trans`) converts cells and documents from one language to another through a sequential workflow:

- **Terminology analysis** — extracts technical terms, entities, and aliases; builds canonical mappings
- **Semantic modeling** — builds a semantic model with sections, entities, actors, actions, constraints, and workflows
- **Multi-variant generation** — produces three translation variants: literal, technical, and AI-optimized
- **Context enrichment** — merges glossary, semantic model, and variants into a unified context
- **Synthesis** — selects the best variant per segment using a decision matrix
- **Validation** — independent quality audit covering structure, semantics, terminology, and AI readability
- **Apply results** — writes synthesized documents to the paths defined in the cell manifest
- **Finalize** — runs linting, fixes errors, and produces a human-readable report

The pipeline automatically detects the source language and asks for the target document/cell path and target language.

It preserves document structure, section order, list order, algorithm order, requirements, constraints, prompt logic, and instruction priority throughout the process. Translating a cell affects only CodeManifest annotations, inline usages, and `.usages/*.md` files.

The pipeline stops on the first error and retries up to 2 times before failing.

See [Translation pipeline](pipeline/index.md).

### Prompt-engineering review

Review pipeline (`goga-tool-scribe-review`) validates prompt documents against established prompt-engineering principles:

- Six independent checks: structure-syntax, prompt-framing, constraint-engineering, entropy-control, semantic-integrity, efficiency
- Conflict resolution across the collected findings
- A planner that builds a fix plan, followed by the fix stage itself
- A validation stage that re-checks the patched documents

For cells, the review affects only CodeManifest annotations and `.usages/*.md` files — code blocks in usage files and backtick-enclosed usage links are strictly preserved.

See [Review pipeline](review/index.md).

## Connecting to an agent

```bash
goga connect <agent>
```
