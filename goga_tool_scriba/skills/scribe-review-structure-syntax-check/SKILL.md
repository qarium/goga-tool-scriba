---
name: goga-tool-scribe-review-structure-syntax-check
description: Goga tool skill — review stage that validates prompt structure and syntax. Detects violations of semantic gradient, continuation pattern suppression, and syntactic anchoring. Consumes documents, produces findings.
---

# structure-and-syntax-check

## Manifest

```yaml
id: goga-tool-scribe-structure-syntax-check

consumes:
  - documents

produces:
  - findings
```

## Objective
Analyze prompt documents and identify violations related to structure and syntax.
Focus on how instructions are organized, sequenced, and structurally represented.
Detect issues that weaken instruction hierarchy, lower instruction adherence in generated output, encourage unwanted continuation patterns, or create structural ambiguity.

Validate the following principles:
- Semantic Gradient
- Continuation Pattern Suppression
- Syntactic Anchoring

Generate findings for each detected violation.
Only report findings.

## Constraints
- Consider CODEMANIFEST annotations (in header, types, methods, properties) as a separate structural unit
- Consider CODEMANIFEST in-line usages as a separate structural unit

## Loop
Execute very rules for 3 iterations:
1. local analysis — analyze the fragment in isolation.
2. context analysis — analyze the fragment in the context of the entire document.
3. conflict analysis — verify that the finding remains valid when considering all surrounding instructions.

## Rules

### Semantic Gradient

#### Metadata

```yaml
name: semantic_gradient
category: structure_syntax
```

#### Instructions
Your task is to detect if a prompt violates the strict linear progression of instructions. 

To prevent cognitive blending, you MUST execute the analysis in exactly two steps:

**STEP 1:** Component Labeling
Analyze the input prompt line by line. Label every sentence/block as one of the following exact components (do not invent other labels):
- [ROLE/CONTEXT]: AI persona definitions, background info, or situational context.
- [CONSTRAINT]: Negative rules, formatting limits, tone requirements, boundaries, or do-not-dos.
- [TASK]: Direct commands, execution verbs (e.g., "Analyze", "Write", "Translate"), or final calls to action.

**STEP 2:** Sequence Validation
Look at the sequence of labels from top to bottom.
The ONLY acceptable sequence is: [ROLE/CONTEXT] -> [CONSTRAINT] -> [TASK].

Report a finding ("REPORT FINDING") ONLY if any of the following strict violations occur:
1. Any [TASK] label appears higher in the text than a [ROLE/CONTEXT] label.
2. Any [CONSTRAINT] label appears lower in the text than a [TASK] label.
3. Constraint blocks are split (a [CONSTRAINT] appears, then a [ROLE/CONTEXT], then another [CONSTRAINT]).

#### Examples

##### Bad

```text
Analyze the API.

Be concise.

Use an engineering style.

You are a technical editor.
```

##### Good

```text
You are a technical editor.

Use an engineering style.

Be concise.

Analyze the API.
```

### Continuation Pattern Suppression

#### Metadata

```yaml
name: continuation_pattern_suppression
category: structure_syntax
```

#### Instructions
Your task is to detect the presence of generic conversational anchor phrases within the operational (active) zones of a prompt. 

To ensure 100% stability, execute your analysis in exactly three steps:

**STEP 1:** Text Zoning & Layering
Analyze the input text and segment it line-by-line (or block-by-block) into exact zones. Label each block with one of the following tags:
- [SAFE_ZONE]: Any text that is inside code blocks (```), inside explicit quotes ("...", '...'), part of a block explicitly labeled as an example, or part of raw source material/data provided for analysis.
- [OPERATIONAL_ZONE]: Any text that represents active instructions, system prompts, tasks, formatting rules, or direct commands to the AI.

**STEP 2:** Exact Phrase Matching
Scan ONLY the text labeled as [OPERATIONAL_ZONE]. Check for the exact presence (case-insensitive) of the following target phrases:
1. "of course"
2. "certainly"
3. "gladly"
4. "happy to help"
5. "let me explain"
6. "please help"
7. "help me with"
8. "provide something useful"
9. "improve this"
10. "without unnecessary explanations"

**STEP 3:** Exception & Task Overrides
Check if the [OPERATIONAL_ZONE] contains an explicit, literal instruction that forces the AI to act as a conversational chatbot (e.g., "act as a conversational assistant", "simulate a casual chat"). 
- If such an explicit conversational task exists, set OVERRIDE = TRUE.
- Otherwise, set OVERRIDE = FALSE.

#### Examples

##### Bad

```text
Without unnecessary explanations, improve the text.
```

##### Good

```text
Output format:
- issue
- cause
- recommendation
```

### Syntactic Anchoring

#### Metadata

```yaml
name: syntactic_anchoring
category: structure_syntax
```

#### Instructions
Your task is to detect "dense prose" — prompts where multiple requirements or formatting rules are buried inside standard text paragraphs instead of being isolated into structured, scannable assets.

To ensure deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Paragraph & Structure Profiling
Analyze the input text and calculate the following metrics:
1. [PROSE_PARAGRAPHS]: Count the number of text paragraphs that contain 2 or more sentences.
2. [STRUCTURAL_ANCHORS]: Count the presence of explicit structural elements: numbered lists (1., 2.), bullet points (-, *), clear headers (e.g., "Requirements:", "Format:"), or data formats (JSON, YAML, XML tags).
3. [TOTAL_COMMANDS]: Count the total number of execution verbs, constraints, or rules embedded across the entire text.

**STEP 2:** Strict Violation Scoring
Scan the [PROSE_PARAGRAPHS] and trigger a violation check ONLY if any of the following technical conditions are met:
- Rule 1 (Embedded Requirements): A single prose paragraph contains 3 or more distinct requirements, instructions, or actions (e.g., "Do X, ensure Y, and output Z" all in one paragraph).
- Rule 2 (Informal Format Block): The text defines an output format (contains keywords like "output format", "return as", "respond with"), but this definition is written inside a standard prose paragraph without using a list, JSON, YAML, or code block.
- Rule 3 (Structure Deficit): The [TOTAL_COMMANDS] is 4 or more, but the [STRUCTURAL_ANCHORS] count is 0. (The prompt is complex but completely flat).

**STEP 3:** Simplicity Exception (Gatekeeper)
Evaluate if the prompt is basic enough to be exempt:
- If the total length of the prompt is 2 sentences or less, AND it contains 2 or fewer instructions -> Set EXEMPT = TRUE.
- Otherwise -> Set EXEMPT = FALSE.

#### Examples

##### Bad

```text
Please follow these requirements. Use JSON output, validate all fields, avoid explanations, and return only the result.
```

##### Good

```text
Requirements:

1. Use JSON output.
2. Validate all fields.
3. Return only the result.
4. Do not include explanations.
```

## Output
For every detected violation append a finding to the `pipeline_context.findings`.

**Output format:**

```yaml
findings:
  - category: structure_syntax
    rule: semantic_gradient
    message: >
      Instruction hierarchy is inverted.
    evidence: >
      Analyze the API. Be concise. Use an engineering style. You are a technical editor.
    recommendation: >
      Reorganize the prompt from role definition to constraints and then to the execution task.
```