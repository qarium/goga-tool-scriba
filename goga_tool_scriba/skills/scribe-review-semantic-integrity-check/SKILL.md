---
name: goga-tool-scribe-review-semantic-integrity-check
description: Goga tool skill — review stage that validates semantic integrity. Detects local semantic density, minimal semantic ambiguity, attention collision, and retrieval affinity violations that reduce model understanding or retrieval precision. Consumes documents, produces findings.
---

# semantic-integrity-check

## Manifest

```yaml
id: goga-tool-scribe-review-semantic-integrity-check

consumes:
  - documents

produces:
  - findings
```

## Objective
Analyze prompt documents and identify violations related to semantic integrity.

Focus on semantic quality (defined below as the four validated principles:
local semantic density, minimal semantic ambiguity, attention collision, and retrieval affinity)
rather than formatting, style, or optimization.

Detect issues that:
- reduce model understanding
- increase semantic ambiguity
- weaken semantic coupling
- cause concept blending
- reduce retrieval precision

Validate the following principles:
- Local Semantic Density
- Minimal Semantic Ambiguity
- Attention Collision
- Retrieval Affinity

Generate findings for each detected violation.
Only report findings.

## Loop
Execute very rules for 3 iterations:
1. local analysis — analyze the fragment in isolation.
2. context analysis — analyze the fragment in the context of the entire document.
3. conflict analysis — verify that the finding remains valid when considering all surrounding instructions.

## Rules

### Local Semantic Density

#### Metadata

```yaml
name: local_semantic_density
category: semantic_integrity
```

#### Instructions
Your task is to detect fragmented instructions — instances where rules or constraints referencing the exact same entity or artifact are unnecessarily separated by unrelated structural content blocks instead of being grouped together.

To ensure 100% deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Entity & Block Mapping
Analyze the input prompt from top to bottom. Identify and number each distinct text block (paragraphs, lists, headers). 
1. [TOTAL_BLOCKS]: Count the total number of structural blocks in the prompt. If [TOTAL_BLOCKS] = 5, AND there is at least one entity pair where IS_FRAGMENTED == TRUE. (This is a direct violation: related instructions are split by noise blocks).
- Rule 2 (Compliant Proximity): All related instructions are either adjacent, properly nested in a list, or the prompt is short ([TOTAL_BLOCKS] < 5).

#### Examples

##### Bad

```text
Use JSON output.

...

Response must be valid.
```

##### Good

```text
Return only valid JSON output.
```

### Minimal Semantic Ambiguity

#### Metadata

```yaml
name: minimal_semantic_ambiguity
category: semantic_integrity
```

#### Instructions
Your task is to detect ambiguous, overloaded, or undefined terms that expand behavioral variance, as well as to catch the mixing of high-level concepts and low-level specifications within a single requirement block.

To ensure 100% deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Ambiguity Marker & Term Extraction
Scan the entire input text and identify all terms or phrases that fall into the following linguistic categories (case-insensitive):
1. [AMBIGUOUS_TOKENS]: "smart", "powerful", "advanced", "flexible", "professional", "high-quality", "robust", "better", "premium", "optimal", "intuitive".
2. [MIXED_ABSTRACTION]: A single sentence or paragraph that combines a high-level goal (e.g., "optimize user experience", "ensure systemic safety") with a granular, low-level instruction (e.g., "add a semicolon", "capitalize the first word") without structural separation.

If zero tokens or mixed structures are found, immediately terminate and output OK.

**STEP 2:** Structural Grounding & Definition Audit
For each element flagged in Step 1, verify if it is structurally anchored or defined. Set IS_DEFINED = TRUE if and only if the term or block satisfies at least one of the following conditions:
1. [EXPLICIT_DEFINITION]: The ambiguous term is immediately followed by a definition block, a colon explanation, or acceptance criteria (e.g., "Use high-quality output: this means zero typos and 100% compliant JSON format").
2. [DOMAIN_RECOGNITION]: The term is part of an established, recognized domain-specific standard or technology specification (e.g., "Robust Estimation in statistics", "Advanced Encryption Standard / AES").
3. [SPECIFIC_INTERPRETATION]: The document has a dedicated glossary or section titled "Definitions" or "Interpretation" that maps the term.

If the term is used in a vacuum as a direct requirement (e.g., "Make the response smart and professional"), set IS_DEFINED = FALSE.

**STEP 3:** Strict Violation Scoring
Evaluate the flagged components using the following strict logical matrix:
- Rule 1 (Floating Ambiguity): A sentence contains a token from [AMBIGUOUS_TOKENS], AND its validation status is IS_DEFINED == FALSE. (This is a direct violation: the AI is given a subjective target with no boundaries).
- Rule 2 (Abstraction Collision): The [MIXED_ABSTRACTION] flag is triggered within the same unseparated instruction block.
- Rule 3 (Compliant Terminology): All potential ambiguities are successfully resolved via Step 2 conditions, meaning IS_DEFINED == TRUE for all of them.

#### Examples

##### Bad

```text
Make the response smart and professional.
```

##### Good

```text
Use a technical and concise style.
```

### Attention Collision

#### Metadata

```yaml
name: attention_collision
category: semantic_integrity
```

#### Instructions
Your task is to detect instruction blocks that introduce too many distinct technical entities (tools, artifacts, components, roles) inside flat prose without using structural separation like lists, tables, or schemas.

To ensure 100% deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Entity Counting & Sentence Profiling
Analyze the input prompt sentence by sentence. For each independent sentence/paragraph, count the number of distinct technical entities introduced. 
- Technical entities include: tools, code components, artifacts, services, roles, outputs, or technical libraries/modules.
- Record the maximum count under [MAX_ENTITIES_IN_PROSE].

If the entire prompt contains fewer than 4 technical entities in total, immediately terminate and output OK.

**STEP 2:** Structural Demarcation Audit
Check how these entities are physically presented in the text. Set IS_STRUCTURED = TRUE if and only if the detected entities satisfy at least one of the following physical conditions:
1. [LIST_STRUCTURE]: The entities are split into separate Markdown list items (using `-`, `*`, `1.`, `2.`, etc.).
2. [TABULAR_STRUCTURE]: The entities are organized inside a Markdown table or explicit schema block (JSON, YAML, XML tags).
3. [EXPLICIT_OWNERSHIP]: Each entity in the sentence is immediately bounded by its own exclusive verb or clear possessive marker that prevents cross-association (e.g., "The matcher does X, the adapter does Y, and the decorator does Z").

If 4 or more entities are listed sequentially inside a single flat sentence using commas or conjunctions (e.g., "A with X, Y, Z and W"), set IS_STRUCTURED = FALSE.

**STEP 3:** Strict Score Evaluation
Evaluate the components from Step 1 and Step 2 against the following strict logical matrix:
- Rule 1 (Entity Overload Violation): [MAX_ENTITIES_IN_PROSE] >= 4, AND the structural classification is IS_STRUCTURED == FALSE. (This is a direct violation: too many concepts are crammed into flat text, creating cognitive blending).
- Rule 2 (Compliant Presentation): All blocks containing 4 or more entities are properly formatted using lists/tables, or the text never exceeds 3 sequential entities.

#### Examples

##### Bad

```text
Python matcher library with fixtures, assertions, adapters and decorators.
```

##### Good

```text
Library components:
- fixtures
- matchers
- adapters
- decorators
```

### Retrieval Affinity

#### Metadata

```yaml
name: retrieval_affinity
category: semantic_integrity
```

#### Instructions
Your task is to detect instances where a prompt uses vague, descriptive, or colloquial phrasing ("how it works") instead of using established, industry-standard canonical technical terms, which reduces precision and retrieval efficiency.

To ensure 100% deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Descriptive Phrasing Detection (Reverse Dictionary Scan)
Analyze the input text line by line. Identify sentences that use non-specific nouns combined with functional descriptions (e.g., "the thing that...", "mechanism for...", "system to do...").
Specifically check if any block matches these behavioral violation archetypes:
1. [THING_PATTERN]: Phrases like "the thing that converts/serializes/saves..."
2. [DESCRIPTIVE_COMPLEX]: Multi-word explanations of basic operations (e.g., "automatic dependency wiring", "checking data structure correctness", "endpoint communication mechanism").

If the text contains exclusively compact technical terms, terminate and output OK.

**STEP 2:** Canonical Mapping & Exemption Check
For every descriptive phrase flagged in Step 1, execute a virtual token compaction step:
- Attempt to map the description to an exact industry standard/canonical term (e.g., "thing that converts objects into text" -> "JSON/XML Serialization"; "automatic dependency wiring" -> "Dependency Injection").
- If a clear canonical term exists, set CANONICAL_EXISTS = TRUE. Otherwise, set FALSE.

Next, verify exemptions. Set IS_EXEMPT = TRUE only if at least one condition is met:
1. [GLOSSARY_EXPLANATION]: The descriptive wording is used *in addition* to the canonical term as an explanation (e.g., "Use Dependency Injection (which means automatic wiring)...").
2. [AUDIENCE_TARGET]: The prompt explicitly states it is designed for beginners, children, or non-technical users (e.g., "Explain to a 5-year old").

If CANONICAL_EXISTS == TRUE and no exemptions match, set IS_EXEMPT = FALSE.

**STEP 3:** Strict Score Evaluation
Evaluate the flags using the following strict logical matrix:
- Rule 1 (Anti-Patterns / Descriptive Vagueness): The text contains a flagged descriptive phrase, CANONICAL_EXISTS == TRUE, and IS_EXEMPT == FALSE. (This is a direct violation: amateur description instead of professional jargon).
- Rule 2 (Compliant Technical Code): The prompt uses precise canonical terminology throughout, or properly introduces explanations.

#### Examples

##### Bad

```text
Use the thing that serializes objects.
```

##### Good

```text
Use deterministic JSON serialization.
```

## Output
For every detected violation append a finding to the `pipeline_context.findings`.

**Output format:**

```yaml
findings:
  - category: semantic_integrity
    rule: minimal_semantic_ambiguity
    message: >
      Ambiguous terminology detected.
    evidence: >
      smart
    recommendation: >
      Replace the ambiguous term with a specific technical characteristic.
```