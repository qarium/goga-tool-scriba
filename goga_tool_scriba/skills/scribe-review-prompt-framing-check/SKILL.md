---
name: goga-tool-scribe-review-prompt-framing-check
description: Goga tool skill — review stage that validates prompt framing. Detects embedding alignment, frame locking, semantic priming, and latent state persistence violations that cause frame drift or behavioral variance. Consumes documents, produces findings.
---

# prompt-framing-check

## Manifest

```yaml
id: goga-tool-scribe-review-prompt-framing-check

consumes:
  - documents

produces:
  - findings
```

## Objective
Analyze prompt documents and identify violations related to prompt framing.
Focus on how the prompt establishes, maintains, and reinforces the intended generation mode.
Detect issues that cause frame drift, latent state contamination, incorrect inference mode activation, or weak behavioral anchoring.

Validate the following principles:
- Embedding Alignment
- Frame Locking
- Semantic Priming
- Latent State Persistence

Generate findings for each detected violation.
Only report findings.

## Loop
Execute very rules for 3 iterations:
1. local analysis — analyze the fragment in isolation.
2. context analysis — analyze the fragment in the context of the entire document.
3. conflict analysis — verify that the finding remains valid when considering all surrounding instructions.

## Rules

### Embedding Alignment

#### Metadata

```yaml
name: embedding_alignment
category: prompt_framing
```

#### Instructions
Your task is to detect the collision of incompatible behavioral modes (e.g., mixing creative/exploratory language within a deterministic/validation workflow).

To ensure deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Vocabulary Extraction & Counting
Scan the entire input text and check for the presence of specific lexical markers (case-insensitive). Count and record how many indicators from each group appear in the text:
1. [DIVERGENT_MARKERS]: "creative", "brainstorm", "imagine", "interesting", "innovative", "explore", "think freely", "out of the box", "novel".
2. [CONSTRAINED_MARKERS]: "strict", "exact", "deterministic", "formal specification", "validate", "verify", "compliant", "precise", "rigid".

**STEP 2:** Workflow Classification
Determine the dominant objective of the prompt based on its primary execution verbs and context:
- If the prompt explicitly asks to generate ideas, brainstorm, or explore concepts -> Set WORKFLOW = EXPLORATORY.
- If the prompt asks to write code, create specifications, validate data, verify facts, or follow strict schemas -> Set WORKFLOW = DETERMINISTIC.

**STEP 3:** Collision Validation Matrix
Check for structural and semantic violations using the following mathematical rules:
- Rule 1 (Mixed Signals Collision): Both [DIVERGENT_MARKERS] count >= 1 AND [CONSTRAINED_MARKERS] count >= 1. (This is a direct conflict of modes, regardless of workflow).
- Rule 2 (Divergent Leakage): WORKFLOW == DETERMINISTIC, but [DIVERGENT_MARKERS] count >= 1. (Creative vocabulary has leaked into a strict task).
- Rule 3 (Brainstorming in Validation): The text contains keywords like "validate" or "verify", but also contains keywords like "brainstorm" or "imagine".

#### Examples

##### Bad

```text
Generate a strict technical specification.
Use creative and innovative approaches.
```

##### Good

```text
Generate a strict technical specification.
Use deterministic reasoning and exact terminology.
```

### Frame Locking

#### Metadata

```yaml
name: frame_locking
category: prompt_framing
```

#### Instructions
Your task is to detect the collision of incompatible operational frames within the same un-demarcated scope (e.g., demanding analytical rigor and emotional entertainment simultaneously without structural boundaries).

To ensure deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Frame Component Identification
Scan the prompt and identify if elements (verbs, adjectives, requirements) trigger any of the following specific frames:
- [ANALYTICAL]: analyze, evaluate, audit, metrics, engineering, formal, architecture.
- [ENTERTAINING]: fun, emotional, hilarious, joke, witty, entertaining, casual.
- [SPECIFICATION]: architecture spec, formal specification, requirements, constraints, RFC.
- [STORYTELLING]: narrative, plot, character, story, lore, emotional arc.
- [DETERMINISTIC]: strict, exact, zero-variance, mathematical, precise.
- [BRAINSTORMING]: ideas, suggestions, concepts, possibilities, think freely.
- [VALIDATION]: verify, audit, test compliance, validate fields, check errors.
- [ROLEPLAY]: act as, pretend to be, adopt persona, impersonate.
- [COMPLIANCE]: legal, regulatory, strict adherence, standard compliance.
- [CREATIVE_EXPLORATION]: innovative, creative, non-standard, novel approaches.

Record all frames that have at least 1 trigger word present.

**STEP 2:** Boundary & Phase Demarcation Check
Analyze the physical structure of the prompt to see if these frames are isolated. Set BOUNDARIES = TRUE if ANY of the following structural conditions are met:
1. The text uses explicit multi-phase headers (e.g., "Phase 1:", "Step 2:").
2. The frames are separated into completely different blocks or code blocks.
3. The prompt contains explicit transition phrases (e.g., "After completing the analysis, switch to...", "Then, in the next phase...").
4. One of the conflicting frames appears strictly inside a section labeled as an Example or Quote.

If none of these conditions are met, set BOUNDARIES = FALSE.

**STEP 3:** Incompatibility Matrix Validation
If BOUNDARIES == FALSE, check for the presence of any of the following exact conflicting pairs (from Step 1):
- [ANALYTICAL] AND [ENTERTAINING]
- [SPECIFICATION] AND [STORYTELLING]
- [DETERMINISTIC] AND [BRAINSTORMING]
- [VALIDATION] AND [ROLEPLAY]
- [COMPLIANCE] AND [CREATIVE_EXPLORATION]

#### Examples

##### Bad

```text
Analyze the architecture in a fun and emotional way.
```

##### Good

```text
Analyze the architecture using a formal engineering approach.
```

### Semantic Priming

#### Metadata

```yaml
name: semantic_priming
category: prompt_framing
```

#### Instructions
Your task is to detect the delayed establishment of execution context (when a Role definition is placed lower in the text than the initial Task or Constraint).

To ensure 100% deterministic evaluation, execute the analysis in exactly three steps:

**STEP 1:** Component & Index Mapping
Analyze the input prompt line by line from top to bottom. Assign a 1-based index (Line 1, Line 2, etc.) to each statement and identify the following components based on specific structural markers:
1. [ROLE_LINE]: Record the line number where a Role Definition first appears. 
   * Markers include: expertise declarations ("expert in", "senior"), persona definitions ("You are a...", "Act as a..."), authority/responsibility statements ("Your responsibility is to...").
   * If NO role definition exists anywhere in the text -> Set [ROLE_LINE] = NULL.
2. [FIRST_ACTION_LINE]: Record the line number where the VERY FIRST task statement or behavioral constraint appears.
   * Markers include: execution verbs ("Analyze", "Write", "Let's think", "Process"), formatting rules, or behavioral boundaries.
3. [TITLE_OR_OPENING]: Check if the role definition appears in the very first sentence or within a designated Markdown header block at the very top (e.g., `# Role`, `## Context`).
   * If yes -> Set [TITLE_OR_OPENING] = TRUE.
   * If no -> Set [TITLE_OR_OPENING] = FALSE.

**STEP 2:** Strict Positional Validation
Compare the extracted indexes using the following strict logical rules:
- Condition 1 (No Role): If [ROLE_LINE] == NULL -> Set VIOLATION = FALSE (No role to validate).
- Condition 2 (Correct Order): If [ROLE_LINE] < [FIRST_ACTION_LINE] OR [TITLE_OR_OPENING] == TRUE -> Set VIOLATION = FALSE (Context established early).
- Condition 3 (Delayed Context): If [ROLE_LINE] > [FIRST_ACTION_LINE] AND [TITLE_OR_OPENING] == FALSE -> Set VIOLATION = TRUE (The role appears chronologically later than the first task/constraint).

#### Examples

##### Bad

```text
Let's think together.

...

You are a senior compiler engineer.
```

##### Good

```text
You are a senior compiler engineer.
Analyze the implementation according to compiler design principles.
```

### Latent State Persistence

#### Metadata

```yaml
name: latent_state_persistence
category: prompt_framing
```

#### Instructions
Your task is to detect when a prompt switches from one behavioral execution mode to an incompatible one without using an explicit reset or transition command ("ungrounded transition").

To ensure 100% deterministic compliance, execute the analysis in exactly three steps:

**STEP 1:** Chronological Mode Mapping
Analyze the input prompt line by line from top to bottom. Identify and list all behavioral execution modes present in the text in their exact chronological order of appearance. Categorize each detected mode into one of the following exact types:
- [CREATIVE_ROLEPLAY]: Prompts involving imaginative, fictional, historical, or casual personas (e.g., "Imagine you are a medieval king", "Act as a pirate").
- [DETERMINISTIC_ANALYSIS]: Prompts involving technical, analytical, strict, or engineering tasks (e.g., "Analyze the API specification", "Verify the code", "Audit this schema").

**STEP 2:** Transition Barrier Scan
Scan the text intervals located strictly *between* the detected modes to check for explicit transition markers. Set TRANSITION_FOUND = TRUE if ANY of the following exact phrases or clear structural equivalents appear between Mode A and Mode B:
1. Commands that terminate a mode (e.g., "End roleplay", "Stop simulation").
2. Commands that explicitly switch focus (e.g., "Switch to technical mode", "Now adopt a formal style", "Change perspective to").
3. Context reset instructions (e.g., "Reset previous assumptions", "Ignore previous persona").
4. Strict scoping (e.g., one of the modes is wrapped inside an explicit block labeled strictly as an Example or Quote).

If no such markers exist between the modes, set TRANSITION_FOUND = FALSE.

**STEP 3:** State Machine Validation
Evaluate the chronological list from Step 1 and the flag from Step 2 against the following strict violation criteria:
- Rule 1 (Ungrounded Category Switch): The prompt establishes a [CREATIVE_ROLEPLAY] mode and later establishes a [DETERMINISTIC_ANALYSIS] mode (or vice-versa), AND TRANSITION_FOUND == FALSE.
- Rule 2 (No Violation/Single Mode): The prompt contains only one execution mode category throughout the entire text, OR multiple modes exist but they belong to the exact same category.

#### Examples

##### Bad

```text
Imagine you are a medieval king.

...

Analyze the API specification.
```

##### Good

```text
Imagine you are a medieval king.

...

End roleplay.
Use a formal technical style.
Analyze the API specification.
```

## Output
For every detected violation append a finding to the `pipeline_context.findings`.

**Output format:**

```yaml
findings:
  - category: prompt_framing
    rule: frame_locking
    message: >
      Conflicting execution frames detected.
    evidence: >
      Analyze the architecture in a fun and emotional way.
    recommendation: >
      Use a single execution frame and remove conflicting behavioral signals.
```