---
name: metabolic-reaction-site-extraction
description: Use when you have a PathModel knowledge base (.lp) with molecules as
  atom/bond predicates and known reactions, and need reaction sites (biochemical transformations)
  to drive analogical inference of new reactions or metabolite structures.
when_to_use_negative:
- When molecules are not (and cannot be) encoded as atom/bond predicates with consistent
  atom numbering — the article notes manual encoding of the starting knowledge base
  and atom numbering during reactions cannot yet be fully automated
- When the goal is genome-scale metabolic network reconstruction rather than analogical
  inference on a manually curated pathway with a defined source and goal molecule
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3391
tools:
- name: PathModel
  role: Python3/ASP package whose ReactionSiteExtraction.lp script performs the reaction-site
    extraction from known reactions before incremental inference with PathModel.lp
  repo: https://github.com/pathmodel/pathmodel
- name: clingo
  role: Grounder and solver executing the ASP scripts (ReactionSiteExtraction.lp,
    MZComputation.lp, PathModel.lp) including incremental mode
  repo: https://github.com/potassco/clingo
- name: clyngor
  role: Python wrapper around clingo used by the PathModel wrapper to run the ASP
    programs
  repo: https://github.com/Aluriak/clyngor
- name: RDKit
  role: Draws the structures of newly inferred metabolites (e.g., MAA1, MAA2) after
    reaction-site-driven inference via pathmodel_plot
  repo: https://github.com/rdkit/rdkit
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:ffb8f48599abe00aa48dd5efc33a3e27fce9ead50dafba00edcfec59c40f3398
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# metabolic-reaction-site-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract biochemical transformations (reaction sites before and after a reaction) from known reactions encoded as atom and bond predicates in an ASP .lp knowledge base, so that PathModel can analogically apply these transformations to infer new reactions and metabolite structures (e.g., in the MAA pathway from sedoheptulose-7-phosphate toward palythine and MAA1).

## When to use

Use when you have a PathModel knowledge base (.lp) with molecules as atom/bond predicates and known reactions, and need reaction sites (biochemical transformations) to drive analogical inference of new reactions or metabolite structures.

## When NOT to use

- When molecules are not (and cannot be) encoded as atom/bond predicates with consistent atom numbering — the article notes manual encoding of the starting knowledge base and atom numbering during reactions cannot yet be fully automated
- When the goal is genome-scale metabolic network reconstruction rather than analogical inference on a manually curated pathway with a defined source and goal molecule

## Inputs

- ASP logic program (.lp) knowledge base with molecules encoded as atom/bond predicates and known reactions (e.g., MAA_pwy.lp, sterol_pwy.lp)
- Metabolomic constraints in the .lp file: mzfiltering values (m/z × 10,000, e.g., 2703000 and 3023117) and absentmolecules (e.g., Z-palythenic acid)
- Source and goal molecule identifiers within the knowledge base

## Outputs

- Reaction sites before and after each known reaction (the biochemical transformation of each reaction)
- pathmodel_data_transformations.tsv listing extracted transformations
- pathmodel_output.lp with newreaction, predictatom, predictbond, reaction, and inferred atoms feeding downstream inference
- Predicted metabolite structures (e.g., MAA1 at m/z 270.3) and inferred reactions consumable by pathmodel_plot

## How to apply

Encode the pathway knowledge base (e.g., MAA_pwy.lp for the Chondrus crispus MAA pathway from sedoheptulose-7-phosphate with goals MAA1 and palythine) with atoms and bonds of each molecule and the known reactions. Run ReactionSiteExtraction.lp, which compares atoms and bonds of reactant and product for each reaction and extracts a 'before' reaction site (atoms/bonds present in the reactant but absent in the product) and an 'after' reaction site (atoms/bonds present in the product but absent in the reactant); these two sites together constitute the biochemical transformation induced by the reaction. This step runs automatically before PathModel.lp when invoking 'pathmodel infer', and alongside MZComputation.lp which computes m/z for each known molecule (m/z values multiplied by 10,000 since clingo does not handle decimals) and m/z changes between reactant and product. The extracted transformations enable the incremental clingo solver to abstract molecular transformations from prior biochemical knowledge and apply them to new substrates. Inspect pathmodel_data_transformations.tsv and pathmodel_output.lp (newreaction, predictatom, predictbond predicates) to verify the transformations extracted and applied.

## Related tools

- **PathModel** (Python3/ASP package whose ReactionSiteExtraction.lp script performs the reaction-site extraction from known reactions before incremental inference with PathModel.lp) — https://github.com/pathmodel/pathmodel
- **clingo** (Grounder and solver executing the ASP scripts (ReactionSiteExtraction.lp, MZComputation.lp, PathModel.lp) including incremental mode) — https://github.com/potassco/clingo
- **clyngor** (Python wrapper around clingo used by the PathModel wrapper to run the ASP programs) — https://github.com/Aluriak/clyngor
- **RDKit** (Draws the structures of newly inferred metabolites (e.g., MAA1, MAA2) after reaction-site-driven inference via pathmodel_plot) — https://github.com/rdkit/rdkit

## Examples

```
pathmodel infer -i MAA_pwy.lp -o output_folder -s 100
```

## Evaluation signals

- pathmodel_data_transformations.tsv contains a before/after site pair for every known reaction in the input .lp
- Incremental inference reaches the goal molecules (palythine, MAA1) from sedoheptulose-7-phosphate within the step limit (-s 100)
- MZ-filtered prediction yields a molecule at m/z 270.3 (mzfiltering(2703000)) matching the measured peak
- pathmodel_output.lp contains newreaction, predictatom, and predictbond predicates for the three putative enzymatic reactions (decarboxylation/dehydration steps)
- Excluded molecule Z-palythenic acid (absentmolecules) does not appear in any inferred transformation

## Limitations

- Pathmodel is a working prototype; scalability of the approach is an open issue
- Manual burden of entering molecular structures has not been minimized and atom numbering during metabolic reactions cannot be fully automated
- Enzymes for porphyra-334 dehydration and decarboxylation are unknown, making the MAA inversion hypothesis not genomically tractable

## Evidence

- [methods] The first one, ReactionSiteExtraction.lp creates biochemical transformation from reactions.: [evidence span withheld — anchor status: not_found]
- [methods] A second script, MZComputation.lp will compute the MZ for each known molecule.: [evidence span withheld — anchor status: not_found]
