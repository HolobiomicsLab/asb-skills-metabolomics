---
name: asp-fact-encoding-of-molecular-structures
description: Use when building or extending a PathModel knowledge base (.lp fact file)
  where each molecule must be represented as numbered atoms and bonds and each reaction
  as a reactant-product pair, prior to extracting biochemical transformations.
when_to_use_negative:
- Structures are not manually curated/numbered yet — atom numbering during metabolic
  reactions cannot be fully automated, so unnumbered or inconsistently numbered structures
  will yield invalid reaction-site diffs
- Input reactions come from databases like MetaCyc without re-numbering atoms in a
  consistent manner across reactions, which PathModel's transformation abstraction
  requires
- The goal is inferring reactions/metabolites from M/Z ratios or genomic data rather
  than abstracting transformations from already-known reactions
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: PathModel
  role: Provides the ASP scripts (ReactionSiteExtraction.lp, MZComputation.lp, PathModel.lp)
    and the .lp knowledge-base format this encoding targets; invoked via the `pathmodel`
    wrapper
  repo: https://github.com/pathmodel/pathmodel
- name: clingo
  role: Grounder and solver that evaluates the encoded ASP facts and rules (requires
    Lua compatibility, e.g. installed via conda)
  repo: https://github.com/potassco/clingo
- name: clyngor
  role: Python wrapper through which clingo is invoked to run ReactionSiteExtraction.lp
    on the encoded fact files
  repo: https://github.com/Aluriak/clyngor
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:cc6cc3d78794fba5c164b4b4a3d48630af06e1b58585c692deca3356c59eb9da
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# asp-fact-encoding-of-molecular-structures

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes known metabolite structures and enzymatic reactions as Answer Set Programming (ASP) facts — atom/bond predicates per molecule and reactant-product links per reaction — so PathModel's ReactionSiteExtraction.lp, run through the clingo grounder/solver, can abstract reactions into molecular transformations for the sterol and MAA pathway knowledge bases.

## When to use

Use when building or extending a PathModel knowledge base (.lp fact file) where each molecule must be represented as numbered atoms and bonds and each reaction as a reactant-product pair, prior to extracting biochemical transformations.

## When NOT to use

- Structures are not manually curated/numbered yet — atom numbering during metabolic reactions cannot be fully automated, so unnumbered or inconsistently numbered structures will yield invalid reaction-site diffs
- Input reactions come from databases like MetaCyc without re-numbering atoms in a consistent manner across reactions, which PathModel's transformation abstraction requires
- The goal is inferring reactions/metabolites from M/Z ratios or genomic data rather than abstracting transformations from already-known reactions

## Inputs

- ASP fact file (.lp) encoding molecules as atom and bond predicates with manual atom numbering (e.g. pathmodel/pathmodel/data/sterol_pwy.lp, pathmodel/pathmodel/data/MAA_pwy.lp, pathmodel_test_data.lp)
- Reaction facts linking each reactant to its product (single-reactant/single-product enzymatic reactions: 15 sterol reactions, 13 MAA reactions)

## Outputs

- ASP predicates diffAtomBeforeReaction, diffAtomAfterReaction, diffBondBeforeReaction, diffBondAfterReaction, siteBeforeReaction, siteAfterReaction from ReactionSiteExtraction.lp
- pathmodel_data_transformations.tsv with one row per distinct biochemical transformation (12 sterol, 9 MAA)

## How to apply

Encode each metabolite in a `.lp` fact file (e.g. sterol_pwy.lp, MAA_pwy.lp) using atom and bond predicates with manually assigned atom numbers, and encode each enzymatic reaction as a link between exactly one reactant and one product; consistent atom numbering across molecules is essential because ReactionSiteExtraction.lp compares atom-by-atom. Run the extraction through clyngor, which invokes the clingo grounder/solver, so the script compares atoms and bonds of the two molecules of each reaction and computes set differences: atoms/bonds present in the reactant but absent in the product (site before) and those present in the product but absent in the reactant (site after), emitted as diffAtomBeforeReaction, diffAtomAfterReaction, diffBondBeforeReaction, diffBondAfterReaction, siteBeforeReaction and siteAfterReaction predicates. Collapse identical before/after site pairs across reactions into distinct molecular transformations and write one row per transformation to pathmodel_data_transformations.tsv. Validate by checking that the abstracted transformation counts match Table S5: 12 transformations from the 15 sterol reactions and 9 from the 13 MAA reactions.

## Related tools

- **PathModel** (Provides the ASP scripts (ReactionSiteExtraction.lp, MZComputation.lp, PathModel.lp) and the .lp knowledge-base format this encoding targets; invoked via the `pathmodel` wrapper) — https://github.com/pathmodel/pathmodel
- **clingo** (Grounder and solver that evaluates the encoded ASP facts and rules (requires Lua compatibility, e.g. installed via conda)) — https://github.com/potassco/clingo
- **clyngor** (Python wrapper through which clingo is invoked to run ReactionSiteExtraction.lp on the encoded fact files) — https://github.com/Aluriak/clyngor

## Examples

```
pathmodel infer -i pathmodel_test_data.lp -o output_folder
```

## Evaluation signals

- Each molecule in the .lp file has a complete set of atom and bond facts with manually assigned, consistent atom numbers across reactant and product
- ReactionSiteExtraction.lp emits the six expected predicates (diffAtomBeforeReaction, diffAtomAfterReaction, diffBondBeforeReaction, diffBondAfterReaction, siteBeforeReaction, siteAfterReaction) for every reaction
- Number of distinct transformations in pathmodel_data_transformations.tsv equals 12 for the 15 sterol reactions and 9 for the 13 MAA reactions, matching Table S5
- Every before/after site pair derives from a valid single-reactant/single-product reaction in the knowledge base
- `pathmodel test -o output_folder` (container smoke test) runs the full encode-extract-infer pipeline without solver errors

## Limitations

- The main limitation is user time necessary to properly encode the starting knowledge base; molecular structures and atom numberings must be entered manually
- It is not yet possible to fully automate the atom numbering during metabolic reactions, so encoding errors in numbering propagate into wrong reaction-site diffs
- MetaCyc encodes reactions in a way that does not make it possible to automatically number atoms from different reactions in the same manner, so database reactions cannot be imported as-is

## Evidence

- [readme] README description of how reactions are compared to extract sites: "the script will compare atoms and bonds of the two molecules of the reaction and will extract a reaction site before the reaction"
- [methods] ASP implementation with clingo: "PathModel is developed in `ASP <https://en.wikipedia.org/wiki/Answer_set_programming>`__ using the `clingo grounder and solver"
- [results] Location of the sterol pathway fact file: "Input data for sterol pathway are in `pathmodel/pathmodel/data/sterol_pwy.lp"
- [discussion] Manual encoding burden of the knowledge base: "The main limitation will be the user time necessary to properly encode the starting knowledge base."
- [supplementary] Transformation yield validation counts: [evidence span withheld — anchor status: not_found]
