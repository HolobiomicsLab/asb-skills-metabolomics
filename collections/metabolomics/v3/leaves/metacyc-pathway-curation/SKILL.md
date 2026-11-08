---
name: metacyc-pathway-curation
description: Use when building a PathModel input knowledge base from MetaCyc pathway
  entries, encoding metabolite structures and enzymatic reactions as ASP facts for
  pathway inference.
when_to_use_negative:
- When the pathway of interest is not covered by existing MetaCyc entries or literature
  reactions, since the knowledge base is manually encoded and cannot be automatically
  generated
- 'When atom-to-atom mapping is required from a database: MetaCyc does not number
  atoms consistently across reactions, so manual curation with consistent numbering
  is required'
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3345
- http://edamontology.org/topic_0602
tools:
- name: PathModel
  role: Consumes the curated .lp knowledge base as input for incremental inference
    of reactions and metabolite structures
  repo: https://github.com/pathmodel/pathmodel
- name: clingo
  role: Grounder/solver used to parse-check and ground the curated ASP facts without
    errors
  repo: https://github.com/potassco/clingo
- name: clyngor
  role: Python wrapper used via the clyngor interface to confirm the .lp facts ground
    without errors
  repo: https://github.com/Aluriak/clyngor
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:9471e70671a327b8021c16447c3a1ac8bb24860893e63cbb3b848dff22c1e1dd
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
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3345
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# metacyc-pathway-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curates MetaCyc pathway definitions (e.g., sterol pathways PWY18C3-1, PWY-2541, PWY66-4 and MAA pathway PWY-7751) into PathModel ASP knowledge-base files (.lp), encoding molecules as atom/bond facts and reactions as reactant-product facts for inference of biochemical reactions and metabolite structures.

## When to use

Use when building a PathModel input knowledge base from MetaCyc pathway entries, encoding metabolite structures and enzymatic reactions as ASP facts for pathway inference.

## When NOT to use

- When the pathway of interest is not covered by existing MetaCyc entries or literature reactions, since the knowledge base is manually encoded and cannot be automatically generated
- When atom-to-atom mapping is required from a database: MetaCyc does not number atoms consistently across reactions, so manual curation with consistent numbering is required

## Inputs

- MetaCyc pathway entries (PWY18C3-1, PWY-2541, PWY66-4, PWY-7751, RXN-17371) with metabolite structures
- Literature-hypothesized reactions (e.g., C3-transfer, orphan MAA reactions with mycosporine-glycine)
- Observed m/z values for predicted metabolites (e.g., MAA1/MAA2)
- List of molecules absent in the study organism (e.g., 22-dehydrocholesterol)

## Outputs

- ASP knowledge-base file (.lp) with atom(), bond(), reaction(), source(), init(), goal(), absentmolecules(), and mzfiltering() facts (e.g., sterol_pwy.lp, MAA_pwy.lp)

## How to apply

Retrieve the MetaCyc definitions and metabolite structures for the relevant pathways (e.g., PWY18C3-1, PWY-2541, PWY66-4 for sterols; PWY-7751 for shinorine biosynthesis plus RXN-17371). Encode each molecule as ASP facts using atom(molecule,id,type) and bond(molecule,bondtype,atom1,atom2), modelling only non-hydrogen atoms and single/double bonds, keeping atom numbering consistent across reactant-product pairs so ReactionSiteExtraction can align transformations. Encode each enzymatic transformation as a reaction(name,reactant,product) fact; declare source() molecules, init()/goal() pathway endpoints, absentmolecules() for orphan molecules to be excluded from inference, and mzfiltering() facts with m/z values multiplied by 10,000 because the solver does not use decimals. Validate by parse-checking the .lp file with clingo (via clyngor) so facts ground without errors and match the deposited reference files (e.g., sterol_pwy.lp with 15 reactions and 24 molecules; MAA_pwy.lp with 13 reactions and 12 molecules).

## Related tools

- **PathModel** (Consumes the curated .lp knowledge base as input for incremental inference of reactions and metabolite structures) — https://github.com/pathmodel/pathmodel
- **clingo** (Grounder/solver used to parse-check and ground the curated ASP facts without errors) — https://github.com/potassco/clingo
- **clyngor** (Python wrapper used via the clyngor interface to confirm the .lp facts ground without errors) — https://github.com/Aluriak/clyngor

## Examples

```
clingo --output-format=0 pathmodel/pathmodel/data/sterol_pwy.lp
```

## Evaluation signals

- The .lp file parses under clingo without errors
- Molecule, reaction, source, and goal facts match the deposited reference file pathmodel/pathmodel/data/sterol_pwy.lp (15 reactions, 24 molecules)
- Molecule, reaction, and mzfiltering facts match pathmodel/pathmodel/data/MAA_pwy.lp (13 reactions, 12 molecules)
- Atom numbering is consistent across reactant-product pairs so biochemical transformations can be extracted
- mzfiltering values are integers (each m/z multiplied by 10,000), not decimals

## Limitations

- Manual encoding of the knowledge base is the main time cost; the burden of manually entering molecular structures has not been minimized
- Atom numbering during metabolic reactions cannot be fully automated
- Scalability is an open issue; PathModel is only a working prototype

## Evidence
