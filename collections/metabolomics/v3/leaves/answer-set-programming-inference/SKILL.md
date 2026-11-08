---
name: answer-set-programming-inference
description: Use when a pathway must be completed by inference from a knowledge base
  of atom/bond-encoded molecules and reactions (e.g. .lp files), given a source metabolite,
  goal molecules, and constraints like m/z filtering or absent molecules.
when_to_use_negative:
- The target pathway already has a complete, well-annotated reaction set with no unassigned
  peaks or missing enzymes — direct database lookup (e.g. MetaCyc) is preferable
- The task requires fully automated atom numbering or scalable genome-scale reconstruction
  — the authors state Pathmodel is only 'a working prototype' and atom numbering during
  reactions 'cannot be fully automated'
- The knowledge base of molecular structures and reactions has not been manually encoded
  — the authors note the main limitation is 'the user time necessary to properly encode
  the starting knowledge base'
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3391
tools:
- name: PathModel
  role: ASP-based inference engine; wrapper (pathmodel_wrapper.py) for the three ASP
    scripts; invoked via 'pathmodel infer' and 'pathmodel test' CLI commands
  repo: https://github.com/pathmodel/pathmodel
- name: clingo
  role: Grounder and solver executing PathModel.lp in incremental mode to compute
    answer sets from the .lp knowledge base
  repo: https://github.com/potassco/clingo
- name: clyngor
  role: Python wrapper around clingo used by the PathModel wrapper; installable with
    the clingo binary via clyngor-with-clingo
  repo: https://github.com/Aluriak/clyngor
- name: RDKit
  role: Cheminformatics library used by the plotting script to draw inferred molecule
    structures (e.g. MAA1, MAA2)
  repo: https://github.com/rdkit/rdkit
- name: networkx
  role: Graph library (with graphviz and pygraphviz) used by pathmodel_plot to render
    the pathway diagram
- name: matplotlib
  role: Plotting package used by the pathmodel_plot rendering pipeline
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:80d06a47748e35c8373fa4fe62589ce74391d1fc4c08cf5e02d76abd09e96827
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
  - RDKit
  - clingo
  - clyngor
  - graphviz
  - matplotlib
  - networkx
  - pygraphviz
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# answer-set-programming-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Skill for driving analogical biochemical inference with Answer Set Programming (ASP) in clingo, as implemented by PathModel: reaction sites are extracted from known reactions, and an incremental solver infers novel reactions and metabolite structures from a source molecule toward goal molecules under metabolomic constraints.

## When to use

Use when a pathway must be completed by inference from a knowledge base of atom/bond-encoded molecules and reactions (e.g. .lp files), given a source metabolite, goal molecules, and constraints like m/z filtering or absent molecules.

## When NOT to use

- The target pathway already has a complete, well-annotated reaction set with no unassigned peaks or missing enzymes — direct database lookup (e.g. MetaCyc) is preferable
- The task requires fully automated atom numbering or scalable genome-scale reconstruction — the authors state Pathmodel is only 'a working prototype' and atom numbering during reactions 'cannot be fully automated'
- The knowledge base of molecular structures and reactions has not been manually encoded — the authors note the main limitation is 'the user time necessary to properly encode the starting knowledge base'

## Inputs

- ASP knowledge base file (.lp) with atom/bond facts encoding molecule structures and reaction/3 facts encoding reactions (e.g. MAA_pwy.lp, sterol_pwy.lp, pathmodel_test_data.lp)
- source molecule declaration and goal molecule declarations
- metabolomic constraint predicates: mzfiltering (m/z × 10,000 as integer) and absentmolecules declarations
- maximal inference step limit (e.g. -s 100)

## Outputs

- pathmodel_output.lp containing newreaction, predictatom, predictbond, reaction, and inferred atom/bond predicates
- pathmodel_data_transformations.tsv listing inferred biochemical transformations
- pathmodel_incremental_inference.tsv tracing the stepwise inference
- pathmodel_output.svg pathway diagram with known (green) and inferred (blue) reactions
- rendered molecule structure images for inferred metabolites (e.g. MAA1, MAA2)

## How to apply

Encode molecules as atom/bond predicates and reactions as reaction/3 facts in a .lp knowledge base (e.g. MAA_pwy.lp or sterol_pwy.lp), declaring source and goal molecules plus constraint predicates: mzfiltering (m/z multiplied by 10,000 since clingo does not handle decimals, e.g. mzfiltering(2703000) for m/z 270.3) and absentmolecules to forbid metabolites not in the organism. Run 'pathmodel infer -i <input.lp> -o output_folder -s 100', which first executes ReactionSiteExtraction.lp (extracting reaction sites before/after each reaction by comparing reactant and product atoms/bonds) and MZComputation.lp (computing m/z for known molecules and m/z changes per reaction), then drives PathModel.lp in clingo's incremental mode, applying two inference methods stepwise from the source until a goal is reached within the maximal step limit (100). Inspect pathmodel_output.lp for newreaction, predictatom, predictbond, reaction, and inferred predicates, and check pathmodel_data_transformations.tsv and pathmodel_incremental_inference.tsv to trace the inferred transformations. Render results with 'pathmodel_plot -i output_folder', which uses RDKit for molecule structures and networkx/graphviz/pygraphviz/matplotlib for the pathway diagram (known reactions in green, newly inferred in blue).

## Related tools

- **PathModel** (ASP-based inference engine; wrapper (pathmodel_wrapper.py) for the three ASP scripts; invoked via 'pathmodel infer' and 'pathmodel test' CLI commands) — https://github.com/pathmodel/pathmodel
- **clingo** (Grounder and solver executing PathModel.lp in incremental mode to compute answer sets from the .lp knowledge base) — https://github.com/potassco/clingo
- **clyngor** (Python wrapper around clingo used by the PathModel wrapper; installable with the clingo binary via clyngor-with-clingo) — https://github.com/Aluriak/clyngor
- **RDKit** (Cheminformatics library used by the plotting script to draw inferred molecule structures (e.g. MAA1, MAA2)) — https://github.com/rdkit/rdkit
- **networkx** (Graph library (with graphviz and pygraphviz) used by pathmodel_plot to render the pathway diagram)
- **matplotlib** (Plotting package used by the pathmodel_plot rendering pipeline)

## Examples

```
pathmodel infer -i MAA_pwy.lp -o output_folder -s 100 && pathmodel_plot -i output_folder
```

## Evaluation signals

- The incremental clingo inference reaches all declared goal molecules (e.g. palythine, MAA1, or 22-dehydrocholesterol/brassicasterol/stigmasterol) within the maximal step limit (100 steps)
- The m/z-filtered prediction yields a molecule compatible with the measured ratio (e.g. m/z 270.3 encoded as mzfiltering(2703000))
- Forbidden metabolites declared with absentmolecules (e.g. ergosterol, fucosterol, zymosterol, Z-palythenic acid) are absent from all inferred paths
- pathmodel_output.lp contains newreaction, predictatom, predictbond, and reaction predicates consistent with pathmodel_data_transformations.tsv and pathmodel_incremental_inference.tsv
- pathmodel_output.svg renders the expected pathway topology matching the article figures (e.g. early vs. late SSR sterol routes of Figure 5)

## Limitations

- Scalability is an open issue; 'The Pathmodel version we present here is a working prototype'
- The starting knowledge base must be manually encoded; 'It is not yet possible to fully automate the atom numbering during metabolic reaction'
- clingo does not handle decimals, so m/z values must be multiplied by 10,000 in the .lp file
- Inference depends on which pathway/branch is selected beforehand: 'The selection of the relevant pathway to be considered ... is therefore a key pre-processing step'

## Evidence

- [readme] PathModel is developed in ASP using the clingo grounder and solver: "PathModel is developed in `ASP <https://en.wikipedia.org/wiki/Answer_set_programming>`__ using the `clingo grounder and solver <https://github.com/potassco/clingo>`__"
- [readme] ReactionSiteExtraction extracts biochemical transformation as atom/bond changes between reactant and product: [evidence span withheld — anchor status: not_found]
- [readme] MZComputation computes m/z for each known molecule and m/z changes per reaction: [evidence span withheld — anchor status: not_found]
- [methods] M/Z ratio multiplied by 10,000 for ASP filtering: "M/Z ratio can be added to check whether there is a metabolite that can be predict with this ratio. M/Z ratio must be multiplied by 10 000"
- [methods] absentmolecules excludes metabolites from inference: "Molecules absent in the organism of study can be specified. They will not be used by the inference method."
- [discussion] Manual encoding of the starting knowledge base is the main limitation: "The main limitation will be the user time necessary to properly encode the starting knowledge base."
- [discussion] Scalability remains an open issue; Pathmodel is a prototype: "The next issue is about the scalability of our approach. The Pathmodel version we present here is a working prototype"
