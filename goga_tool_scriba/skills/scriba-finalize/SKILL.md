---
name: goga-tool-scriba-finalize
description: Finalize the translation pipeline — run linting across all cells, fix errors, and produce a human-readable report of the entire pipeline output
---

# save-finalize-pipeline

## Objective
Final verify and report.

## Steps
1. Run bash command `goga lint` without arguments for all cells validation
2. **MUST** fix lint errors
3. Produces results to `pipeline_context.finish_status`
4. Print final report

## Final Report

**Report template**

```markdown
## Languages
{{ pipeline_context.source.language | human_readable }}

## Glossary
{{ pipeline_context.glossary | human_readable }}

## Semantic Model
{{ pipeline_context.semantic_model | human_readable }}

## Enriched Context
{{ pipeline_context.enriched_context | human_readable }}

## Validation
{{ pipeline_context.validation | human_readable }}

## Documents Translated
{{ pipeline_context.translated_files | human_readable }}

## Status
{{ pipeline_context.finish_status | human_readable }}
```

**Rules**

`human_readable` function is making format data from pipeline context for human-readable report:
- **Format**: Use Markdown. Apply clear heading hierarchies (#, ##, ###), bold text for key metrics, and bullet points for lists.
- **Key Translation**: Convert technical camelCase or snake_case keys into natural human language (e.g., change "auth_attempts_total" to "Total Authentication Attempts").
- **Data Formatting**:
  - Format timestamps into a readable date-time format (e.g., YYYY-MM-DD HH:MM).
  - Format large numbers with commas (e.g., 1,250,000).
  - Convert boolean values (true/false) into clear status words (e.g., "Enabled", "Active", "Yes" / "Disabled", "Inactive", "No") depending on the context.
- **Handling Missing Data**: If a value is null, empty, or missing, display it as "N/A" or "Not provided". Do not omit the key unless it is completely irrelevant.
- **Tone**: Maintain a professional, objective, and technical-oriented tone. Avoid developer jargon.