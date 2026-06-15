---
name: goga-tool-scriba-trans-apply-results
description: Save synthesized translated documents to files at the paths defined in the cell manifest
---

# save-translated-files

## Objective
Save results from synthesized documents to files.

## Manifest
```yaml
id: goga-tool-scriba-trans-apply-results

consumes:
  - source.documents
  - synthesized_documents

produces:
  - translated_files
```

## Action
Save synthesized documents

## Output Contract
```yaml
- saved/file/path_1
- saved/file/path_2
```