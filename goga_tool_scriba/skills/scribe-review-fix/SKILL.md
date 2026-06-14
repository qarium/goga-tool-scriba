---
name: goga-tool-scribe-review-fix
description:
---

# prompt-engineering-fixer

## Manifest

```yaml
id: goga-tool-scribe-review-fix

consumes:
  - documents
  - fix_plan

produces:
  - fixed_documents
```

## Objective
Resolve all reported findings by applying targeted text modifications that address the specific violation described in each finding.

## Constraints

1. Apply only modifications directly justified by findings
2. Preserve all existing requirements and the original business objective unchanged
3. Modify only the text fragment directly referenced by the finding, without altering surrounding context
4. **No New Requirements (anti-injection)**: A fix MUST NOT introduce requirements, constraints, behavioral contracts,
   or implementation mechanisms that are not already present in the original document. When a finding flags an undefined or ambiguous term,
   the fix MUST either rephrase to surface the ambiguity as an explicit open question for the author (e.g., "Define the tool error representation.")
   OR leave the fragment unchanged and mark the finding as **unresolved — requires author decision**.
   The fix MUST NEVER select one concrete interpretation among several valid ones and write it into the document.
   Selecting an interpretation is an authoring decision, not a review decision.

## Fixing Principles

### Preserve Intent
The original purpose of the prompt must remain unchanged.

### Preserve Constraints
Valid constraints **MUST** be saved.

### No New Requirements (anti-injection)
A fix MUST NOT introduce requirements, constraints, behavioral contracts, or implementation mechanisms
that are not already present in the original document. When a finding flags an undefined or ambiguous term,
the fix MUST either rephrase to surface the ambiguity as an explicit open question for the author,
OR leave the fragment unchanged and mark the finding as **unresolved — requires author decision**.
The fix MUST NEVER select one concrete interpretation among several valid ones and write it into the document.

### Preserve Structure
Work only with the documents from `pipeline_context`.
Modify only the text fragment directly referenced by the finding, without altering surrounding context.

### Explicit Consent                                                                                                                                                                                
A fix MUST NOT be applied silently. Before any document modification, present the proposed change                                                                                                   
to the user, explain what it does and why, and wait for explicit approval. Only after approval may                                                                                                  
the change be written to disk.  

## Workflow
1. Read documents.
2. Read `pipeline_context.fix_plan`.
3. **Explain and confirm each fix individually.** For every planned fix, present to the user:                                                                                                       
   - **Location**: document path and line number(s) of the fragment to change.                                                                                                                      
   - **Finding**: the finding ID, category, and rule this fix resolves.                                                                                                                             
   - **Evidence**: the exact source text the finding cited.                                                                                                                                         
   - **Before**: the current fragment, quoted verbatim.                                                                                                                                             
   - **After**: the proposed fragment, quoted verbatim.                                                                                                                                             
   - **Rationale**: 1-3 sentences explaining why this change resolves the finding, why it                                                                                                           
     preserves intent, and (when relevant) why it does not violate anti-injection.                                                                                                                  
   Then ask the user whether to apply this specific change.                                                                                                                                         
4. **Apply only approved fixes**, one at a time, in save order (modifications that shift line                                                                                                       
   numbers are applied bottom-up to keep earlier references stable).                                                                                                                                
5. For every rejected fix, leave the fragment unchanged and mark the corresponding finding                                                                                                          
   as **unresolved — requires author decision**.                                                                                                                                                    
6. Verify that each applied fix does not introduce contradictions with adjacent instructions                                                                                                        
   or break references within the same document section.                                                                                                                                            
7. Produce corrected documents.    

## Output
Collect all fixed documents append to the `pipeline_context.fixed_documents`

```yaml
fixed_documents:
  - path: <document>
    changes:
      - status: applied | rejected                                                                                                                                                                  
        location: <file:lines>                                                                                                                                                                      
        before: |                                                                                                                                                                                   
          <original fragment>                                                                                                                                                                       
        after: |                                                                                                                                                                                    
          <new fragment — present for applied changes; equals 'before' for rejected>                                                                                                                
        rationale: |                                                                                                                                                                                
          <why this change resolves the finding>   
```