---
name: asp-knowledge-base-encoding
description: 'Use when preparing a PathModel input knowledge base: you need to encode
  molecule structures (atom/bond facts), reactions, source/goal endpoints, and m/z
  or absent-molecule filters as ASP facts.'
when_to_use_negative:
- Input is already structured reaction/metabolite data that another tool (e.g. MetaCyc
  flat files) can serve directly without manual atom numbering
- The goal is pathway visualization or inference itself rather than building the input
  knowledge base — use PathModel's 'pathmodel infer' and 'pathmodel_plot' instead
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0602
tools:
- name: PathModel
  role: Target system whose ASP fact predicates (atom, bond, reaction, source, goal,
    mzfiltering, absentmolecules) the knowledge base must satisfy
  repo: https://github.com/pathmodel/pathmodel
- name: clingo
  role: Grounder and solver used to parse-check that the encoded .lp facts ground
    without errors
  repo: https://github.com/potassco/clingo
- name: clyngor
  role: Python wrapper around clingo used via clyngor-with-clingo to confirm the facts
    parse correctly
  repo: https://github.com/Aluriak/clyngor
- name: MetaCyc
  role: Source of pathway definitions and metabolite structures to be re-encoded as
    ASP facts
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:d065d3b596a575b77d97b9556d5818eea1cc7d7e7f153f2cb0a271fd1c6e27b4
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
  - MetaCyc
  - PathModel
  - clingo
  - clyngor
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# asp-knowledge-base-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encode prior biochemical knowledge (molecule structures and enzymatic reactions from MetaCyc pathways) as ASP facts for PathModel's clingo solver, e.g. sterol_pwy.lp with 15 reactions/24 molecules and MAA_pwy.lp with 13 reactions/12 molecules.

## When to use

Use when preparing a PathModel input knowledge base: you need to encode molecule structures (atom/bond facts), reactions, source/goal endpoints, and m/z or absent-molecule filters as ASP facts.

## When NOT to use

- Input is already structured reaction/metabolite data that another tool (e.g. MetaCyc flat files) can serve directly without manual atom numbering
- The goal is pathway visualization or inference itself rather than building the input knowledge base — use PathModel's 'pathmodel infer' and 'pathmodel_plot' instead

## Inputs

- MetaCyc pathway definitions (e.g. PWY-7751, PWY18C3-1, PWY-2541, PWY66-4) with metabolite structures
- Literature-hypothesized reactions (e.g. RXN-17371 C3-transfer, orphan MAA reactions)
- Measured metabolite m/z values (e.g. MAA1/MAA2) and list of absent molecules

## Outputs

- ASP knowledge base file (e.g. sterol_pwy.lp, MAA_pwy.lp) in pathmodel/pathmodel/data/
- clingo-parseable facts: atom(), bond(), reaction(), source(), init(), goal(), mzfiltering(), absentmolecules()

## How to apply

Retrieve MetaCyc definitions for the relevant pathways (e.g. PWY18C3-1, PWY-2541, PWY66-4 for sterols; PWY-7751 for shinorine). Encode each metabolite with atom(molecule,id,type) and bond(molecule,bondtype,atom1,atom2) facts, modelling only non-hydrogen atoms and single/double bonds, and keep atom numbering consistent across reactant-product pairs. Encode each biochemical transformation as reaction(name,reactant,product) facts. Declare the source molecule with source(), define pathway endpoints with init(pathway(...)) and goal(pathway(...)) targeting the unproducible sterols or MAA goals, mark excluded orphans (e.g. 22-dehydrocholesterol) with absentmolecules(), and add measured masses as mzfiltering facts multiplied by 10,000 because the solver does not use decimals. Finally, parse-check the .lp file with clingo (via clyngor) and verify its molecule, reaction, source, and goal facts against the deposited reference file in pathmodel/pathmodel/data/.

## Related tools

- **PathModel** (Target system whose ASP fact predicates (atom, bond, reaction, source, goal, mzfiltering, absentmolecules) the knowledge base must satisfy) — https://github.com/pathmodel/pathmodel
- **clingo** (Grounder and solver used to parse-check that the encoded .lp facts ground without errors) — https://github.com/potassco/clingo
- **clyngor** (Python wrapper around clingo used via clyngor-with-clingo to confirm the facts parse correctly) — https://github.com/Aluriak/clyngor
- **MetaCyc** (Source of pathway definitions and metabolite structures to be re-encoded as ASP facts)

## Examples

```
clingo --const datadir=pathmodel/data sterol_pwy.lp   # then: pathmodel infer -i pathmodel/data/sterol_pwy.lp -o output_folder -s 100
```

## Evaluation signals

- The .lp file parses under clingo (via clyngor) without grounding errors
- Molecule count matches expectation: 24 molecules/15 reactions for sterol_pwy.lp, 12 molecules/13 reactions for MAA_pwy.lp
- source(), init(), and goal() facts target the eight unproducible sterols; orphan 22-dehydrocholesterol appears only in an absentmolecules() fact
- mzfiltering facts are integers equal to m/z * 10,000
- Molecule, reaction, source, and goal facts match the deposited reference file pathmodel/pathmodel/data/sterol_pwy.lp or MAA_pwy.lp; atom numbering is consistent across reactant-product pairs

## Limitations

- Manual encoding burden: 'the user time necessary to properly encode the starting knowledge base' is the main limitation; atom numbering during metabolic reactions cannot be fully automated
- The solver does not use decimals, so m/z values must be multiplied by 10,000 into integers
- MetaCyc encodes reactions in a way that does not allow automatically numbering atoms identically across different reactions, motivating manual consistent numbering

## Evidence

- [methods] Modelling molecules: "M/Z ratio can be added to check whether there is a metabolite that can be predict with this ratio. M/Z ratio must be multiplied by 10 000"
- [readme] ASP implementation: [evidence span withheld — anchor status: not_found]
