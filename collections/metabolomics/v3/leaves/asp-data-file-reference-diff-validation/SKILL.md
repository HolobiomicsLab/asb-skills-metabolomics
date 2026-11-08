---
name: asp-data-file-reference-diff-validation
description: Use when a Pathmodel/ASP input datafile (e.g. MAA_pwy.lp, sterol_pwy.lp)
  has been manually encoded or edited and must be verified to ground cleanly and match
  a reference before running inference.
when_to_use_negative:
- When the goal is to run PathModel inference itself (ReactionSiteExtraction, MZComputation,
  PathModel.lp) rather than validate the input datafile
- When the knowledge base is generated automatically and already matches the deposited
  reference — this skill targets manually encoded files
- 'When data must contain decimals: mzfiltering facts cannot hold decimal m/z values,
  they must be pre-multiplied by 10,000'
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3345
tools:
- name: clingo
  role: Grounder/solver used to parse-check that the ASP datafile facts ground without
    errors
  repo: https://github.com/potassco/clingo
- name: clyngor
  role: Python wrapper around clingo used to invoke the parse-check of the produced
    MAA_pwy.lp
  repo: https://github.com/Aluriak/clyngor
- name: Pathmodel
  role: Tool whose input datafile format (.lp with molecule, reaction, mzfiltering
    facts) the validation targets; reference file lives in pathmodel/pathmodel/data
  repo: https://github.com/pathmodel/pathmodel
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:8c969d768281bfdb9e2e9106d810bb00acc0fd70198ee449c803138a3caf64ce
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pathmodel/pathmodel
  repo_url: https://github.com/potassco/clingo
  tools:
  - Pathmodel
  - clingo
  - clyngor
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ASP data-file reference diff validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates a hand-encoded ASP knowledge base (e.g. Pathmodel's MAA_pwy.lp with 13 reactions and 12 molecules) by parse-checking its molecule, reaction, and mzfiltering facts with clingo/clyngor and diffing against the deposited reference file. Ensures the knowledge base grounds without errors before inference is run.

## When to use

Use when a Pathmodel/ASP input datafile (e.g. MAA_pwy.lp, sterol_pwy.lp) has been manually encoded or edited and must be verified to ground cleanly and match a reference before running inference.

## When NOT to use

- When the goal is to run PathModel inference itself (ReactionSiteExtraction, MZComputation, PathModel.lp) rather than validate the input datafile
- When the knowledge base is generated automatically and already matches the deposited reference — this skill targets manually encoded files
- When data must contain decimals: mzfiltering facts cannot hold decimal m/z values, they must be pre-multiplied by 10,000

## Inputs

- Pathway knowledge source (MetaCyc entry PWY-7751 shinorine biosynthesis; extended reaction RXN-17371; literature-hypothesized reactions)
- Molecular structures for the 12 MAA pathway molecules (atoms and bonds)
- M/Z ratio values for MAA1/MAA2 metabolites
- Deposited reference ASP datafile (pathmodel/pathmodel/data/MAA_pwy.lp)

## Outputs

- Valid ASP-syntax knowledge base file MAA_pwy.lp in pathmodel/data containing molecule, reaction, and mzfiltering facts
- Clingo/clyngor parse-check confirmation (facts ground without errors)

## How to apply

Collect pathway knowledge (e.g. MetaCyc PWY-7751 shinorine biosynthesis, the C3-transfer reaction RXN-17371, and literature-hypothesized orphan MAA/mycosporine-glycine reactions), then encode each molecule as ASP facts with numbered atoms (hydrogen excluded) and single/double bonds, keeping atom numbering consistent between reactant and product of each reaction. Encode each transformation as a reaction fact linking named reactant and product molecules. Encode MAA1/MAA2 m/z values as mzfiltering facts, multiplying each m/z by 10,000 to yield integers because the solver does not handle decimals. Parse-check the file with clingo via the clyngor Python interface to confirm all facts ground without errors, and finally diff the produced .lp file against the deposited reference (pathmodel/pathmodel/data/MAA_pwy.lp) to confirm facts match exactly.

## Related tools

- **clingo** (Grounder/solver used to parse-check that the ASP datafile facts ground without errors) — https://github.com/potassco/clingo
- **clyngor** (Python wrapper around clingo used to invoke the parse-check of the produced MAA_pwy.lp) — https://github.com/Aluriak/clyngor
- **Pathmodel** (Tool whose input datafile format (.lp with molecule, reaction, mzfiltering facts) the validation targets; reference file lives in pathmodel/pathmodel/data) — https://github.com/pathmodel/pathmodel

## Examples

```
python -c "from clyngor import solve; answers = solve(files=['MAA_pwy.lp']); print(list(answers))"
```

## Evaluation signals

- MAA_pwy.lp exists and is in valid ASP syntax (clingo grounding via clyngor raises no errors)
- The file contains exactly 13 reaction facts linking reactant-product molecule pairs
- Molecule facts cover 12 molecules described by numbered atoms (hydrogen excluded) and single/double bonds, with atom numbering consistent across reactant-product pairs
- mzfiltering facts hold integer values equal to m/z * 10,000 for MAA1/MAA2
- Diff of the produced file against pathmodel/pathmodel/data/MAA_pwy.lp shows matching molecule, reaction, and mzfiltering facts

## Limitations

- The starting knowledge base must be manually encoded; the article notes the main limitation is user time to properly encode the starting knowledge base, and atom numbering during metabolic reactions cannot be fully automated
- clingo does not use decimals, so all m/z values must be pre-multiplied by 10,000 — forgetting this yields silently wrong mzfiltering facts
- Pathmodel is only a working prototype and validation by diff checks syntax/facts, not biochemical correctness of the encoded hypotheses

## Evidence

- [results] MAA_pwy.lp reference file location: "Input data for MAA pathway are in `pathmodel/pathmodel/data/MAA_pwy.lp"
- [methods] ASP implementation with clingo: [evidence span withheld — anchor status: not_found]
- [methods] mzfiltering scaling requirement: "M/Z ratio must be multiplied by 10 000"
- [methods] clyngor wrapper: "clyngor package (can be installed with clingo with clyngor-with-clingo package"
- [discussion] manual encoding burden: "The main limitation will be the user time necessary to properly encode the starting knowledge base."
