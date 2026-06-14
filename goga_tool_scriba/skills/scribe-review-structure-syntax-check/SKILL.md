---
name: goga-tool-scribe-review-structure-syntax-check
description:
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

Operate in read-only mode on all documents.
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
Detect prompts that violate logical instruction progression.
Instructions should establish context before introducing constraints and constraints before introducing tasks.

Report a finding when:
- tasks appear before role definition;
- constraints appear after execution instructions;
- context is introduced after the task;
- instruction hierarchy appears inverted;
- execution flow requires the reader to reconstruct intent.

Prefer the following progression:
1. Role
2. Behavioral mode
3. Constraints
4. Task

If constraint blocks appear both before and after context/role, report a finding — this is a structural split, not a minor ordering difference.

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
Detect conversational assistant phrases that may activate generic assistant response patterns.

Report a finding when ANY of the following phrases appear as operational instructions:
- of course
- certainly
- gladly
- happy to help
- let me explain
- please help
- help me with
- provide something useful
- improve this
- without unnecessary explanations

Report a finding only when the phrase influences execution behavior rather than appearing inside examples or quoted text.

Do not report findings when ANY of the following hold:
- the phrase appears inside an example;
- the phrase appears inside quoted content;
- the task explicitly requires conversational behavior;
- the phrase is part of analyzed source material.

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
Detect prompts that rely solely on prose when structured syntax would increase instruction adherence in generated output.
Language models respond strongly to recognizable structural patterns.

Report a finding when:
- multiple requirements are embedded in long paragraphs;
- output formats are described informally;
- rules are expressed as prose instead of structured lists;
- complex instructions lack structural anchors.

Prefer explicit structures such as:
- numbered lists;
- bullet lists;
- YAML;
- JSON;
- RFC-style sections;
- formal schemas.

Do not report findings when the prompt is sufficiently simple that additional structure would not reduce reader lookup effort.

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