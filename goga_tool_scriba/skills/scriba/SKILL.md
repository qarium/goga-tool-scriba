---
name: goga-tool-scriba
description: Goga tool skill — top-level dispatcher for the Scriba suite. Detects operation intent (translation or review) from the request and dispatches to goga-tool-scriba-trans or goga-tool-scribe-review. Does not collect inputs or own pipeline state — the dispatched skill handles its own onboarding.
---

# Scriba Dispatcher

## System Role
Scriba Suite Dispatcher.
Single responsibility: classify the operation and invoke the matching pipeline skill.

## Objective
Dispatch to exactly one skill per invocation.

## Intent Detection
Classify `$OPERATION` from request signals:

| Signal in request                                                                               | `$OPERATION`  |
|-------------------------------------------------------------------------------------------------|---------------|
| "translate", "перевести", "translation", target language mentioned, source/target language pair | `translation` |
| "review", "проверить", "ревью", "prompt engineering check", "validate prompts", "find issues"   | `review`      |
| Both intents explicitly requested                                                               | `ask`         |
| Ambiguous or unstated                                                                           | `ask`         |

On `ask` — ask user to confirm operation before dispatching.

## Routing

```yaml
translation: goga-tool-scriba-trans
review: goga-tool-scribe-review
```

## Workflow
1. Detect `$OPERATION` from request.
2. If `ask` — confirm operation with user.
3. Invoke the matching skill via Skill tool with the original user request.
4. Return the dispatched skill's output unchanged.

## Output
Echo the dispatched skill's final status and report.