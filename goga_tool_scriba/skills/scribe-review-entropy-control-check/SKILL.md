---
name: goga-tool-scribe-review-entropy-control-check
description: Goga tool skill — review stage that validates entropy control. Detects entropy shaping, distribution steering, and anti-pattern suppression violations that expand the solution space or weaken behavioral predictability. Consumes documents, produces findings.
---

# entropy-control-check

## Manifest

```yaml
id: goga-tool-scribe-review-entropy-control-check

consumes:
  - documents

produces:
  - findings
```

## Objective
Analyze prompt documents and identify violations related to entropy control.
Focus on language that unnecessarily expands the model's solution space, increases output variance, or weakens behavioral predictability.
Detect issues that introduce excessive ambiguity, broaden generation trajectories, or reduce controllability.

Validate the following principles:
- Entropy Shaping
- Distribution Steering
- Anti-pattern Suppression

Generate findings for each detected violation.
Only report findings.

## Loop
Execute rules for 3 iterations:
1. literal analysis — evaluate only explicit document content.
2. semantic analysis — evaluate meaning using document context.
3. adversarial analysis — attempt to disprove the finding and retain only supported findings.

## Rules

### Entropy Shaping

#### Metadata

```yaml
name: entropy_shaping
category: entropy_control
```

#### Instructions
Your task is to detect abstract optimization goals that lack explicit, measurable success criteria or quantitative boundaries.

To ensure 100% deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Optimization Verb Detection
Scan the entire input text and identify sentences containing any of the following optimization verbs (case-insensitive):
- [OPTIMIZATION_VERBS]: "improve", "optimize", "enhance", "refine", "strengthen", "increase quality", "make better", "upgrade".

If zero optimization verbs are detected, immediately terminate and output OK.

**STEP 2:** Measurability & Boundary Audit
For each sentence flagged in Step 1, perform a strict syntactic audit to find explicit metrics. Set HAS_METRIC = TRUE if and only if the sentence (or its immediate bullet-point children) contains at least one of the following elements:
1. [NUMERIC_TARGETS]: Quantitative boundaries using digits or percentages (e.g., "by 30%", "under 500 words", "maximum 3 paragraphs").
2. [STRUCTURAL_CRITERIA]: Negative or positive compliance boundaries that are binary and verifiable (e.g., "without removing technical terms", "using only RFC-8254 standards", "must contain 5 fields").
3. [X_BY_Y_FORM]: Explicit formulation of how success is evaluated (e.g., "Optimize X by checking Y").

If the sentence only contains the verb and an abstract noun (e.g., "Improve the prompt", "Optimize the text", "Enhance the readability") without any constraints from the list above, set HAS_METRIC = FALSE.

**STEP 3:** Strict Score Evaluation
Evaluate the components from Step 1 and Step 2 against the following logical matrix:
- Rule 1 (Vague Optimization Deficit): A sentence contains a verb from [OPTIMIZATION_VERBS], AND its validation status is HAS_METRIC == FALSE. (This is a direct violation: an optimization task is given blindly without success criteria).
- Rule 2 (Compliant Target): All optimization verbs in the text are accompanied by verified targets, meaning HAS_METRIC == TRUE for all of them.

#### Examples

##### Bad

```text
Improve the prompt.
```

##### Good

```text
Reduce prompt length by 30% without removing technical terminology.
```

### Distribution Steering

#### Metadata

```yaml
name: distribution_steering
category: entropy_control
```

#### Instructions
Your task is to detect unanchored absolute requirements — prompts that demand a guaranteed, flawless, or perfect outcome without providing any supporting constraints, examples, or structural boundaries to achieve it.

To ensure 100% deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Absolute Target Scanning
Scan the entire input text and identify if any sentences trigger the presence of absolute outcome demands. Check for the exact presence or clear semantic equivalents of the following phrases (case-insensitive):
- [ABSOLUTE_TARGETS]: "perfect answer", "guarantee correctness", "best solution", "flawless output", "always get it right", "ensure 100% accuracy", "without errors".

If zero absolute targets are found, immediately terminate the analysis and output OK.

**STEP 2:** Support Asset Inventory
Analyze the structural composition of the entire prompt and check for the presence of supporting assets. Calculate the following flags:
1. [HAS_EXAMPLES]: Set to TRUE if the prompt contains a dedicated block labeled as an example (e.g., "Example:", "## Examples", or text inside explicit sample blocks). Otherwise, set to FALSE.
2. [HAS_CONSTRAINTS]: Set to TRUE if the prompt contains a structured list or specific boundary rules (e.g., "under 200 words", "use JSON format", "follow steps 1-3"). Otherwise, set to FALSE.
3. [HAS_CRITERIA]: Set to TRUE if the text explicitly lists acceptance or validation parameters (e.g., "Acceptance criteria:", "The output is valid only if..."). Otherwise, set to FALSE.

**STEP 3:** Structural Anchor Validation
Evaluate the extracted flags from Step 1 and Step 2 using the following strict algorithmic rules to trigger a violation:
- Rule 1 (Unanchored Absolute Demand): At least one phrase from [ABSOLUTE_TARGETS] is present, AND ALL supporting asset flags are FALSE (`[HAS_EXAMPLES] == FALSE` AND `[HAS_CONSTRAINTS] == FALSE` AND `[HAS_CRITERIA] == FALSE`). This is an absolute requirement floating in a vacuum.
- Rule 2 (Anchored Execution): An absolute target is present, but at least ONE supporting asset flag is TRUE (`[HAS_EXAMPLES] == TRUE` OR `[HAS_CONSTRAINTS] == TRUE` OR `[HAS_CRITERIA] == TRUE`). The demand is supported by structure.

#### Examples

##### Bad

```text
Always generate the perfect answer.
```

##### Good

```text
Generate an answer that satisfies all listed constraints and follows the provided examples.
```

### Anti-pattern Suppression

#### Metadata

```yaml
name: anti_pattern_suppression
category: entropy_control
```

#### Instructions
Your task is to detect vague, marketing-oriented, or emotionally loaded descriptors within prompt instructions that increase semantic variance and reduce controllability.

To ensure 100% deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Subjective Token Extraction
Scan the entire input text and identify all sentences containing any of the following subjective or emotional descriptors (case-insensitive):
- [SUBJECTIVE_TOKENS]: "creative", "interesting", "deep", "powerful", "innovative", "nice", "better", "amazing", "impressive", "world-class", "beautiful", "elegant", "smart".

If zero subjective tokens are found, immediately terminate the analysis and output OK.

**STEP 2:** Objective Definition & Task Context Audit
For each sentence flagged in Step 1, perform a strict syntactic audit to check for anchoring constraints or intentional creative tasks. Set IS_ANCHORED = TRUE if and only if the sentence (or its immediate bullet-point children) satisfies at least one of the following conditions:
1. [METRIC_ANCHOR]: The subjective term is immediately backed by numbers, physical metrics, or binary verifiable rules (e.g., "Provide an innovative solution that minimizes latency below 50ms", "Write an interesting text containing exactly 3 facts").
2. [INTENTIONAL_CREATIVE_TASK]: The explicit objective of the prompt is to generate creative fiction, poetry, marketing slogans, or brainstorming options where subjective vocabulary is the required output medium (e.g., "Write an amazing story about a dragon").

If the subjective term is used as a core requirement for a technical, analytical, or functional task without any metrics (e.g., "Provide a powerful and innovative solution", "Make the code better"), set IS_ANCHORED = FALSE.

**STEP 3:** Strict Score Evaluation
Evaluate the flagged elements using the following strict logical matrix:
- Rule 1 (Vague Descriptor Violation): A sentence contains a token from [SUBJECTIVE_TOKENS], AND its validation status is IS_ANCHORED == FALSE. (This is a direct violation: an abstract emotional word is used as a technical requirement).
- Rule 2 (Compliant Usage): All subjective descriptors in the text are properly anchored or used in valid creative workflows, meaning IS_ANCHORED == TRUE for all of them.

#### Examples

##### Bad

```text
Provide a powerful and innovative solution.
```

##### Good

```text
Provide a solution that minimizes latency and reduces memory consumption.
```

## Output
For every detected violation append a finding to the `pipeline_context.findings`.

**Output format:**

```yaml
findings:
  - category: entropy_control
    rule: entropy_shaping
    message: >
      Vague optimization objective detected.
    evidence: >
      Improve the prompt.
    recommendation: >
      Replace the objective with a measurable and verifiable outcome.
```