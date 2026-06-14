---
name: goga-tool-scribe-review-semantic-integrity-check
description:
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
Detect semantically related instructions that are unnecessarily separated.

Report a finding when ALL of the following hold:
- two or more instructions reference the same entity, output, constraint, artifact, or deliverable;
- at least one unrelated structural block exists between them;
- the instructions are not part of an explicit parent-child hierarchy;
- moving the instructions into adjacent positions would reduce cross-reference resolution effort.

Examples of related elements:
- output definition + output constraints;
- artifact definition + artifact requirements;
- task definition + task limitations;
- schema definition + schema validation rules.

Do not report findings when ANY of the following hold:
- the document contains fewer than 5 structural blocks;
- the separation is caused by explicit section hierarchy;
- the related instructions already belong to the same section;
- the separation improves readability without introducing additional lookup effort.

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
Detect ambiguous, subjective, overloaded, or undefined terminology.

Report a finding when ANY of the following hold:
- a term is subjective and lacks measurable criteria;
- a term could reasonably refer to more than one technical property;
- a term is not defined anywhere in the document;
- multiple abstraction levels are mixed within a single requirement.

Examples of commonly ambiguous terms:
- smart
- powerful
- advanced
- flexible
- professional
- high-quality
- robust
- better

Do not report findings when ANY of the following hold:
- the term is explicitly defined nearby;
- the term is a recognized domain-specific term;
- the term has associated acceptance criteria;
- the document defines the expected interpretation.

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
Detect instruction fragments that introduce too many entities without sufficient structural separation.

Report a finding when ALL of the following hold:
- a sentence, paragraph, or instruction introduces 4 or more distinct entities;
- the entities are not represented as a list, table, schema, or structured block;
- attributes, constraints, or responsibilities could plausibly be associated with multiple entities;
- entity boundaries are not explicitly defined.

Examples of entities:
- tools;
- artifacts;
- services;
- components;
- roles;
- outputs.

Do not report findings when ANY of the following hold:
- the entities are represented using structured syntax;
- the entities are organized into separate list items;
- the sentence contains fewer than 4 entities;
- entity ownership is explicitly defined.

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
Detect terminology that reduces retrieval precision by replacing established technical terms with descriptive language.

Report a finding when ALL of the following hold:
- a canonical technical term exists;
- the document uses descriptive wording instead of the canonical term;
- the descriptive wording refers to the same concept;
- replacing the wording with the canonical term would improve precision.

**Examples:**

  Canonical:
  - JSON serialization
  - dependency injection
  - REST API
  - schema validation

  Violations:
  - thing that converts objects into text
  - automatic dependency wiring
  - endpoint communication mechanism
  - checking data structure correctness

Do not report findings when ANY of the following hold:
- the audience requires simplified language;
- the canonical term is immediately introduced afterward;
- the descriptive wording exists solely as an explanation;
- no established canonical term exists.

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