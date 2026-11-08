---
name: metabolite-structure-atom-numbering
description: Use when manually building a PathModel knowledge base (.lp file) that
  requires metabolite structures to be represented as ASP facts with atoms and bonds
  so reactions can be compared at the atom level.
when_to_use_negative:
- When a tool-generated or standardized structure format (e.g. SMILES parsed by RDKit)
  is sufficient — PathModel's ASP encoding requires manual atom numbering and this
  burden is not yet minimized
- When fully automated atom numbering during metabolic reactions is required; the
  article states this is not yet possible to fully automate
- When the pathway is too large for manual encoding — scalability is an open issue
  and PathModel is only a working prototype
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3314
tools:
- name: PathModel
  role: ASP framework whose ReactionSiteExtraction.lp script consumes the atom/bond
    facts and extracts biochemical transformations from reactions
  repo: https://github.com/pathmodel/pathmodel
- name: clingo
  role: Grounder and solver used to parse-check that the encoded atom, bond, and reaction
    facts ground without errors
  repo: https://github.com/potassco/clingo
- name: clyngor
  role: Python wrapper around clingo used to interface with the ASP solver for validation
  repo: https://github.com/Aluriak/clyngor
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:01cf4042ea4e3943621b7ceab9e74fa1c0a59cea4f5f5f1f0137fb169184b593
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pathmodel/pathmodel
  repo_url: https://github.com/pathmodel/pathmodel
  tools:
  - PathModel
  - clingo
  - clyngor
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabolite-structure-atom-numbering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes metabolite structures as ASP facts for PathModel using atom(molecule,id,type) and bond(molecule,bondtype,atom1,atom2) predicates, keeping atom numbering consistent across reactant-product pairs so ReactionSiteExtraction can compare atoms and bonds and extract biochemical transformations.

## When to use

Use when manually building a PathModel knowledge base (.lp file) that requires metabolite structures to be represented as ASP facts with atoms and bonds so reactions can be compared at the atom level.

## When NOT to use

- When a tool-generated or standardized structure format (e.g. SMILES parsed by RDKit) is sufficient — PathModel's ASP encoding requires manual atom numbering and this burden is not yet minimized
- When fully automated atom numbering during metabolic reactions is required; the article states this is not yet possible to fully automate
- When the pathway is too large for manual encoding — scalability is an open issue and PathModel is only a working prototype

## Inputs

- Molecule structures from MetaCyc pathway entries (e.g. PWY-7751, PWY18C3-1, PWY-2541, PWY66-4) or literature
- Biochemical reaction definitions linking reactant and product molecules by name
- Reference knowledge base files (MAA_pwy.lp, sterol_pwy.lp) in pathmodel/pathmodel/data/

## Outputs

- ASP fact files (.lp) with atom(molecule,id,type) and bond(molecule,bondtype,atom1,atom2) predicates
- reaction(name,reactant,product) facts in the same knowledge base file

## How to apply

For each molecule, emit atom facts with numbered non-hydrogen atoms and bond facts restricted to single and double bonds. Critically, keep the atom numbering consistent between the reactant and the product of each reaction, because PathModel's ReactionSiteExtraction script compares atoms and bonds of the two molecules of a reaction to extract reaction sites before and after the transformation. Draw molecule structures and reaction definitions from source databases such as MetaCyc (e.g. PWY-7751 shinorine biosynthesis, PWY18C3-1, PWY-2541, PWY66-4) and literature-hypothesized reactions. When structures cannot be automatically aligned (as in MetaCyc, whose encoding does not allow consistent atom numbering across reactions), number atoms manually. Validate the resulting .lp file by parse-checking with clingo (via clyngor) to confirm the facts ground without errors.

## Related tools

- **PathModel** (ASP framework whose ReactionSiteExtraction.lp script consumes the atom/bond facts and extracts biochemical transformations from reactions) — https://github.com/pathmodel/pathmodel
- **clingo** (Grounder and solver used to parse-check that the encoded atom, bond, and reaction facts ground without errors) — https://github.com/potassco/clingo
- **clyngor** (Python wrapper around clingo used to interface with the ASP solver for validation) — https://github.com/Aluriak/clyngor

## Examples

```
clingo --output-format=sat pathmodel/data/MAA_pwy.lp pathmodel/asp/ReactionSiteExtraction.lp
```

## Evaluation signals

- The produced .lp file parses under clingo without grounding errors
- Atom numbering is consistent across reactant-product pairs for every reaction fact, allowing ReactionSiteExtraction to produce one site before and one site after each reaction
- Molecule facts contain only non-hydrogen atoms and only single/double bond types
- Molecule, reaction, source, init, goal, and mzfiltering facts match the deposited reference files (pathmodel/pathmodel/data/MAA_pwy.lp with 13 reactions/12 molecules; sterol_pwy.lp with 15 reactions/24 molecules)

## Limitations

- Atom numbering during metabolic reactions cannot be fully automated, so encoding remains a manual, time-consuming step (the main limitation is user time to properly encode the starting knowledge base)
- MetaCyc encodes molecules in a way that does not allow atoms from different reactions to be numbered in the same manner automatically
- Scalability of the approach is an open issue; PathModel is only a working prototype

## Evidence
