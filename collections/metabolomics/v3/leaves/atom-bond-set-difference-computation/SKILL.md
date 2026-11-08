---
name: atom-bond-set-difference-computation
description: Use when reactions are encoded as ASP facts (atom/ground predicates with
  reactant-product links, e.g. sterol_pwy.lp, MAA_pwy.lp) and distinct before/after
  reaction sites must be abstracted into reusable molecular transformations.
when_to_use_negative:
- Reactions have multiple reactants or products — the extraction is defined for single-reactant/single-product
  reactions
- The goal is pathway inference or M/Z-based metabolite prediction rather than transformation
  cataloging (use PathModel.lp or MZComputation.lp workflows instead)
- Atom numbering across reactions must be fully automated from structures — the article
  states this cannot yet be fully automated and requires manual knowledge-base encoding
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3172
tools:
- name: PathModel
  role: Package whose ReactionSiteExtraction.lp script performs the atom/bond set-difference
    extraction of biochemical transformations from reactions
  repo: https://github.com/pathmodel/pathmodel
- name: clingo
  role: ASP grounder and solver that executes ReactionSiteExtraction.lp (must be installed
    with Lua compatibility)
  repo: https://github.com/potassco/clingo
- name: clyngor
  role: Python wrapper used to invoke clingo for running the ReactionSiteExtraction
    ASP program
  repo: https://github.com/Aluriak/clyngor
- name: clyngor-with-clingo
  role: Installs clyngor together with a clingo binary so no separate clingo installation
    is needed
  repo: https://github.com/aluriak/clyngor-with-clingo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:9a81557f22b9723156b89c09dfeb636b487f0d43249c96a7f439ee57b1fdc69f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# atom-bond-set-difference-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts biochemical transformations (reaction sites) from enzymatic reactions encoded as ASP facts by computing atom and bond set differences between each reactant and product, yielding a siteBefore/siteAfter pair per reaction — the core abstraction step of PathModel's ReactionSiteExtraction.lp.

## When to use

Use when reactions are encoded as ASP facts (atom/ground predicates with reactant-product links, e.g. sterol_pwy.lp, MAA_pwy.lp) and distinct before/after reaction sites must be abstracted into reusable molecular transformations.

## When NOT to use

- Reactions have multiple reactants or products — the extraction is defined for single-reactant/single-product reactions
- The goal is pathway inference or M/Z-based metabolite prediction rather than transformation cataloging (use PathModel.lp or MZComputation.lp workflows instead)
- Atom numbering across reactions must be fully automated from structures — the article states this cannot yet be fully automated and requires manual knowledge-base encoding

## Inputs

- ASP fact file encoding molecules as atom/ground predicates (e.g. pathmodel/data/sterol_pwy.lp)
- ASP fact file with reactant-product reaction links (e.g. pathmodel/data/MAA_pwy.lp)
- ReactionSiteExtraction.lp ASP program from pathmodel/asp/

## Outputs

- diffAtomBeforeReaction / diffAtomAfterReaction predicates (atoms removed from or added to each reaction site)
- diffBondBeforeReaction / diffBondAfterReaction predicates (bond changes per reaction)
- siteBeforeReaction / siteAfterReaction predicates (complete before/after reaction sites)
- pathmodel_data_transformations.tsv with one row per distinct molecular transformation

## How to apply

Load the pathway ASP fact file and restrict the computation to single-reactant/single-product reactions. For each reaction, take the set difference of atom and bond sets: atoms/bonds present in the reactant but absent in the product form the site before the reaction, and those present in the product but absent in the reactant form the site after. Run this as the ASP program ReactionSiteExtraction.lp through the clingo grounder/solver invoked via clyngor, which emits diffAtomBeforeReaction, diffAtomAfterReaction, diffBondBeforeReaction, diffBondAfterReaction, siteBeforeReaction and siteAfterReaction predicates. Collapse identical before/after site pairs across different reactions to obtain the distinct biochemical transformation catalog, and write one row per transformation to pathmodel_data_transformations.tsv. The rationale is that a transformation abstracts away the specific molecules, allowing identical substructure changes from different reactions to be grouped and later reused for inference.

## Related tools

- **PathModel** (Package whose ReactionSiteExtraction.lp script performs the atom/bond set-difference extraction of biochemical transformations from reactions) — https://github.com/pathmodel/pathmodel
- **clingo** (ASP grounder and solver that executes ReactionSiteExtraction.lp (must be installed with Lua compatibility)) — https://github.com/potassco/clingo
- **clyngor** (Python wrapper used to invoke clingo for running the ReactionSiteExtraction ASP program) — https://github.com/Aluriak/clyngor
- **clyngor-with-clingo** (Installs clyngor together with a clingo binary so no separate clingo installation is needed) — https://github.com/aluriak/clyngor-with-clingo

## Examples

```
pathmodel infer -i sterol_pwy.lp -o output_folder -s 100
```

## Evaluation signals

- Transformation yield matches Table S5: 12 distinct transformations abstracted from the 15 sterol reactions and 9 from the 13 MAA reactions
- Every siteBeforeReaction atom/bond is present in the reactant and absent in the product; every siteAfterReaction atom/bond is present in the product and absent in the reactant (set-difference invariant)
- Identical before/after site pairs from different reactions are collapsed into a single row in pathmodel_data_transformations.tsv (row count equals distinct transformation count, not reaction count)
- All six predicate families (diffAtom/diffBond × before/after, siteBefore/siteAfter) are emitted in the clingo answer set
- Running the shipped test data (`pathmodel test`) reproduces expected transformations without solver errors

## Limitations

- PathModel is an early-state working prototype; scalability of the abstraction step remains an open issue
- Atom numbering during metabolic reactions cannot be fully automated, and manual encoding of the starting knowledge base (structures as atom/bond facts) is a significant time cost
- The abstraction discards molecule context, so transformations capture only substructure changes — downstream selection of the relevant predicted pathway still requires metabolomic/genomic pre-processing

## Evidence

- [methods] ReactionSiteExtraction.lp creates biochemical transformation from reactions: [evidence span withheld — anchor status: not_found]
