---
name: metabolic-transformation-deduplication
description: Use when you have reactions encoded as reactant-product links over atom/bond
  predicates (e.g. sterol_pwy.lp, MAA_pwy.lp) and need distinct molecular transformations
  rather than per-reaction sites.
when_to_use_negative:
- Input reactions lack explicit atom/bond-level structural encoding (e.g. only names
  or SMILES without atom numbering), since the set-difference reasoning requires atom/ground
  predicates.
- The goal is reaction-level inference from a source to a goal molecule with M/Z constraints
  — that is handled by MZComputation.lp and PathModel.lp, not the site-extraction
  step.
- Reactions have multiple reactants or multiple products, where a simple reactant-vs-product
  set difference does not define a single reaction site.
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3345
tools:
- name: PathModel
  role: Prototype package whose ReactionSiteExtraction.lp ASP script performs the
    reaction-site extraction and transformation deduplication
  repo: https://github.com/pathmodel/pathmodel
- name: clingo
  role: ASP grounder and solver that evaluates ReactionSiteExtraction.lp (must be
    installed with Lua compatibility)
  repo: https://github.com/potassco/clingo
- name: clyngor
  role: Python wrapper through which the clingo solver is invoked for the ASP extraction
    scripts
  repo: https://github.com/Aluriak/clyngor
- name: clyngor-with-clingo
  role: pip-installable bundle that ships a clingo binary alongside clyngor so no
    separate clingo install is needed
  repo: https://github.com/aluriak/clyngor-with-clingo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:7fba3963a32675403c7f699a143e59843e79611539bb6d45e81486caca117430
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
  - clyngor-with-clingo
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metabolic-transformation-deduplication

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Abstracts enzymatic reactions in an ASP-encoded pathway knowledge base into a deduplicated catalog of biochemical transformations (before/after reaction-site pairs of changed atoms and bonds), as implemented by PathModel's ReactionSiteExtraction.lp.

## When to use

Use when you have reactions encoded as reactant-product links over atom/bond predicates (e.g. sterol_pwy.lp, MAA_pwy.lp) and need distinct molecular transformations rather than per-reaction sites.

## When NOT to use

- Input reactions lack explicit atom/bond-level structural encoding (e.g. only names or SMILES without atom numbering), since the set-difference reasoning requires atom/ground predicates.
- The goal is reaction-level inference from a source to a goal molecule with M/Z constraints — that is handled by MZComputation.lp and PathModel.lp, not the site-extraction step.
- Reactions have multiple reactants or multiple products, where a simple reactant-vs-product set difference does not define a single reaction site.

## Inputs

- ASP fact file of the pathway (e.g. sterol_pwy.lp, MAA_pwy.lp) encoding molecules as atom/bond predicates and reactions as reactant-product links
- ReactionSiteExtraction.lp ASP program from pathmodel/asp/

## Outputs

- pathmodel_data_transformations.tsv with one row per distinct biochemical transformation
- ASP predicates: diffAtomBeforeReaction, diffAtomAfterReaction, diffBondBeforeReaction, diffBondAfterReaction, siteBeforeReaction, siteAfterReaction

## How to apply

Load the pathway ASP fact files, where molecules are encoded with atom/ground predicates and reactions as reactant-product links. For each reaction, compute a set difference between the atom and bond sets of the reactant and product: atoms/bonds present in the reactant but absent in the product form the 'site before', and those present in the product but absent in the reactant form the 'site after'. Run the extraction as an ASP program (ReactionSiteExtraction.lp) through the clingo grounder/solver invoked via clyngor, emitting diffAtomBeforeReaction, diffAtomAfterReaction, diffBondBeforeReaction, diffBondAfterReaction, siteBeforeReaction and siteAfterReaction predicates. Collapse identical before/after site pairs across different reactions so that the same biochemical transformation shared by multiple reactions is counted once. Write one row per distinct transformation to pathmodel_data_transformations.tsv.

## Related tools

- **PathModel** (Prototype package whose ReactionSiteExtraction.lp ASP script performs the reaction-site extraction and transformation deduplication) — https://github.com/pathmodel/pathmodel
- **clingo** (ASP grounder and solver that evaluates ReactionSiteExtraction.lp (must be installed with Lua compatibility)) — https://github.com/potassco/clingo
- **clyngor** (Python wrapper through which the clingo solver is invoked for the ASP extraction scripts) — https://github.com/Aluriak/clyngor
- **clyngor-with-clingo** (pip-installable bundle that ships a clingo binary alongside clyngor so no separate clingo install is needed) — https://github.com/aluriak/clyngor-with-clingo

## Examples

```
pathmodel infer -i sterol_pwy.lp -o output_folder  # emits pathmodel_data_transformations.tsv among outputs
```

## Evaluation signals

- Transformation yield matches the reported counts: 12 distinct transformations from the 15 sterol reactions and 9 from the 13 MAA reactions (Table S5).
- Each emitted siteBeforeReaction has a corresponding siteAfterReaction for the same reaction, and identical before/after pairs across different reactions are collapsed to a single transformation row.
- pathmodel_data_transformations.tsv contains exactly one row per distinct transformation, with no duplicate site-pair rows.
- Diff predicates (diffAtom*/diffBond*) are consistent with the atom and bond set differences between each reactant and product pair in the input .lp facts.

## Limitations

- PathModel is a working prototype with scalability concerns; the extraction is not guaranteed to generalize to very large reaction networks.
- Manual encoding burden is high: molecular structures must be entered by hand with atom numbering, and atom numbering during metabolic reactions cannot be fully automated.
- MetaCyc encodes reactions in a way that does not allow automatically numbering atoms identically across different reactions, complicating cross-database abstraction.

## Evidence

- [readme] ReactionSiteExtraction creates two sites per reaction: [evidence span withheld — anchor status: not_found]
- [results] Counts of abstracted transformations: [evidence span withheld — anchor status: not_found]
- [results] Output file naming: [evidence span withheld — anchor status: not_found]
