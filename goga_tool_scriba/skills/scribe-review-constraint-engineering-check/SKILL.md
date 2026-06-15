---
name: goga-tool-scribe-review-constraint-engineering-check
description: Goga tool skill — review stage that validates constraint engineering. Detects token inertia, negative instruction, constraint compatibility, and lexical determinism violations that reduce constraint compliance or introduce ambiguity. Consumes documents, produces findings.
---

# constraint-engineering-check

## Manifest

```yaml
id: goga-tool-scribe-review-constraint-engineering-check

consumes:
  - documents

produces:
  - findings
```

## Objective
Analyze prompt documents and identify violations related to constraint engineering.
Focus on how requirements, limitations, and behavioral constraints are defined, prioritized, and reinforced.
Detect issues that reduce constraint compliance, create conflicting requirements, produce non-deterministic output for identical inputs, or make instructions ambiguous to the executing agent.

Validate the following principles:
- Token Inertia
- Negative Instruction
- Constraint Compatibility
- Lexical Determinism

Generate findings for each detected violation.
Only report findings.

## Loop
Execute rules for 3 iterations:
1. literal analysis — evaluate only explicit document content.
2. semantic analysis — evaluate meaning using document context.
3. adversarial analysis — attempt to disprove the finding and retain only supported findings.

## Rules

### Token Inertia

#### Metadata

```yaml
name: token_inertia
category: constraint_engineering
```

#### Instructions
Your task is to detect critical constraints (formatting, compliance, safety) that are positioned too far structurally or textually from the final execution task, increasing the risk of instruction omission.

To ensure 100% deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Anchor Point & Metric Identification
Analyze the input prompt from top to bottom and map the exact locations of the following core elements:
1. [CRITICAL_CONSTRAINT]: Any rule specifying output formats (e.g., "Do not use markdown", "Return JSON"), content restrictions, or safety/compliance boundaries. Record the line number of its first appearance.
2. [FINAL_TASK]: The ultimate command, execution verb, or call to action that triggers the generation (e.g., "Generate the final response", "Write the essay"). Record its line number.
3. [WORD_DISTANCE]: Calculate the exact number of words located strictly between the [CRITICAL_CONSTRAINT] and the [FINAL_TASK].
4. [INTERVENING_SECTIONS]: Count how many distinct Markdown headers (#, ##, ###) or completely separate content topics exist between the constraint and the task.

**STEP 2:** Distance & Reinforcement Validation
Evaluate the metrics from Step 1 against the following strict algorithmic rules to trigger a violation:
- Rule 1 (Textual Distance Deficit): [WORD_DISTANCE] > 150 words, AND the exact same constraint is NEVER repeated or reinforced within 50 words before the [FINAL_TASK].
- Rule 2 (Structural Section Split): [INTERVENING_SECTIONS] >= 2 (meaning the constraint and the task are separated by two or more structural blocks), AND the constraint is not reinforced at the end.
- Rule 3 (Short Prompt Exception): The total word count of the entire prompt is less than 100 words -> Set EXEMPT = TRUE. (In short prompts, instruction decay is mathematically impossible).

#### Examples

##### Bad

```text
Do not use markdown.

...

[many sections later]

Generate the final response.
```

##### Good

```text
Generate the final response.

Do not use markdown.
```

### Negative Instruction

#### Metadata

```yaml
name: negative_instruction
category: constraint_engineering
```

#### Instructions
Your task is to detect prompts that rely heavily on negative phrasing ("what not to do") instead of positive, directive phrasing ("what to do").

To ensure deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Negation Marker Extraction
Analyze the input prompt and extract all sentences containing explicit grammatical negation tokens. Count and record these under two specific categories:
1. [STANDARD_NEGATIONS]: Count sentences containing absolute negation words (case-insensitive):
   * "do not", "don't", "never", "no ", "stop", "avoid", "refrain from", "prohibit".
2. [EXEMPT_NEGATIONS]: Count sentences where the negation token is explicitly linked to a safety, legal, data-privacy, or regulatory boundary (e.g., "Do not leak API keys", "Do not violate copyright", "Never share PII data").

**STEP 2:** Structural Verification Logic
Evaluate the extracted markers from Step 1 against the following rules to trigger a structural violation:
- Rule 1 (Pure Negation): A sentence contains a token from [STANDARD_NEGATIONS] but does NOT provide an immediate, positive alternative or specific target metrics in the same block (e.g., "Do not write long text" vs "Do not write long text, keep it under 3 sentences").
- Rule 2 (Negation Chaining): The total count of [STANDARD_NEGATIONS] across the prompt is 2 or more, creating a chain of prohibitions without positive reinforcement.
- Rule 3 (Exemption Override): If a negation belongs to [EXEMPT_NEGATIONS], it is completely subtracted from the violation score. It is treated as valid compliance code.

#### Examples

##### Bad

```text
Do not use complex sentences.
```

##### Good

```text
Use short sentences containing 5 to 12 words.
```

### Constraint Compatibility

#### Metadata

```yaml
name: constraint_compatibility
category: constraint_engineering
```

#### Instructions
Your task is to detect mutually exclusive or competing constraints within a prompt that demand opposite outcomes simultaneously without establishing a clear hierarchy or order of priority.

To ensure 100% deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Constraint Token Extraction
Analyze the input prompt and check for the presence of specific keywords or semantic concepts. Record which of the following tags are activated:
- [BREVITY]: "brief", "concise", "short", "summary", "minimalist".
- [COMPLETENESS]: "exhaustive", "highly detailed", "comprehensive", "all-inclusive", "thorough".
- [DETERMINISM]: "strict", "exact", "deterministic", "compliant", "formal".
- [CREATIVITY]: "creative", "brainstorm", "innovative", "unrestricted exploration", "freely".

**STEP 2:** Priority Hierarchy Check
Scan the text for an explicit structural block that ranks or prioritizes instructions. Set HAS_PRIORITY = TRUE only if at least one of the following conditions is met:
1. The text contains an explicit ranking list (e.g., "Priority 1:", "Priority 2:" or "Order of importance:").
2. The prompt uses conditional hierarchy phrases (e.g., "Even if it means sacrificing completeness, keep it brief", "X is more important than Y").

If no such ranking structure exists, set HAS_PRIORITY = FALSE.

**STEP 3:** Conflict Matrix Validation
If HAS_PRIORITY == FALSE, evaluate the activated tags from Step 1 against the following exact conflict pairs:
- Pair 1 (The Density Conflict): Both [BREVITY] AND [COMPLETENESS] are activated.
- Pair 2 (The Logic Conflict): Both [DETERMINISM] AND [CREATIVITY] are activated.
- Pair 3 (The Multi-Objective Blur): Three or more distinct tags from Step 1 are activated at the same time without any priority anchors.

#### Examples

##### Bad

```text
Be brief, exhaustive, and highly detailed.
```

##### Good

```text
Priority:

1. Accuracy
2. Brevity
3. Completeness
```

### Lexical Determinism

#### Metadata

```yaml
name: lexical_determinism
category: constraint_engineering
```

#### Instructions
Your task is to detect the presence of weak, non-committal, or probabilistic language within active prompt instructions that causes behavioral variance.

To ensure 100% deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Weak Marker Extraction
Scan the entire input text and identify all sentences containing any of the following specific weak tokens (case-insensitive):
- [WEAK_TOKENS]: "try", "preferably", "maybe", "could", "if possible", "ideally", "should probably".

Record the line numbers where these tokens appear. If zero weak tokens are found, terminate and output OK.

**STEP 2:** Optionality & Context Filtering
For each sentence flagged in Step 1, verify if the weak token is legitimately used for an optional feature. Set IS_EXEMPT = TRUE only if the sentence meets at least one of the following structural conditions:
1. The weak token appears inside an explicit text block designated strictly as an Example or Quote.
2. The sentence explicitly contains an "If/Then" conditional branch that describes an optional feature or secondary choice (e.g., "If you have extra time, try to add a summary", "If formatting fails, maybe return a fallback").

If the weak token is used as a primary constraint or general task guidance (e.g., "Try to follow the spec", "Ideally, use JSON"), set IS_EXEMPT = FALSE.

**STEP 3:** Strict Violation Scoring
Evaluate the flagged elements using the following strict logical matrix:
- Rule 1 (Weak Core Command): A sentence contains a [WEAK_TOKENS] marker, AND its structural classification is IS_EXEMPT == FALSE. (This is a direct violation where a rule or task is weakened by lexical choice).
- Rule 2 (No Violation): All identified weak tokens have IS_EXEMPT == TRUE, OR no weak tokens exist in the text.

#### Examples

##### Bad

```text
Try to follow the specification.
```

##### Good

```text
Follow the specification exactly.
```

## Output
For every detected violation append a finding to the `pipeline_context.findings`.

**Output format:**

```yaml
findings:
  - category: constraint_engineering
    rule: constraint_compatibility
    message: >
      Conflicting constraints detected.
    evidence: >
      Be brief, exhaustive, and highly detailed.
    recommendation: >
      Define explicit constraint priorities and remove incompatible requirements.
```