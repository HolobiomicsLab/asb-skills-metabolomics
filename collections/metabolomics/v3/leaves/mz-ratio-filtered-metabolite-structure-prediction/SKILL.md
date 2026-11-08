---
name: mz-ratio-filtered-metabolite-structure-prediction
description: Use when an unknown metabolite peak (e.g., m/z = 270.3 from metabolomic
  profiling of the MAA pathway) needs a candidate structure inferred from known reactions,
  atom/bond encodings, and mass constraints.
when_to_use_negative:
- The target molecule is a known compound already in the knowledge base — MZComputation.lp
  computes m/z for known molecules, so inference-based structure prediction is unnecessary
- The knowledge base has not been manually encoded as an .lp file with atom/bond predicates
  and numbered atoms for the relevant pathway (Pathmodel cannot yet automate atom
  numbering during reactions)
- No mass-spectrometric m/z constraint or goal molecule is available to anchor the
  incremental inference
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PathModel
  role: ASP pipeline whose PathModel.lp performs incremental m/z-constrained inference
    of new reactions and metabolite structures from source to goal
  repo: https://github.com/pathmodel/pathmodel
- name: clingo
  role: Grounder/solver providing the incremental mode used by PathModel.lp; m/z values
    are integer-scaled ×10,000 because clingo does not handle decimals
  repo: https://github.com/potassco/clingo
- name: clyngor
  role: Python wrapper around clingo used by the pathmodel wrapper to run the ASP
    scripts
  repo: https://github.com/Aluriak/clyngor
- name: RDKit
  role: Cheminformatics library used by pathmodel_plot to draw the predicted molecule
    structures from inferred atoms and bonds
  repo: https://github.com/rdkit/rdkit/
- name: networkx
  role: Graph library (with graphviz and pygraphviz) used to render the inferred pathway
    diagram with known reactions in green and new reactions in blue
  repo: https://github.com/pathmodel/pathmodel
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:f9ca4094ba8f8fbab36368cee5104e8ad5d9ae3e4469e82fbaf274274c04c363
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
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# mz-ratio-filtered-metabolite-structure-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A PathModel/ASP-based method that predicts a putative metabolite structure for an unassigned LC-MS peak by constraining incremental pathway inference with an observed m/z value (e.g., 270.3 for MAA1 in Chondrus crispus) and excluded ('absent') molecules such as Z-palythenic acid.

## When to use

Use when an unknown metabolite peak (e.g., m/z = 270.3 from metabolomic profiling of the MAA pathway) needs a candidate structure inferred from known reactions, atom/bond encodings, and mass constraints.

## When NOT to use

- The target molecule is a known compound already in the knowledge base — MZComputation.lp computes m/z for known molecules, so inference-based structure prediction is unnecessary
- The knowledge base has not been manually encoded as an .lp file with atom/bond predicates and numbered atoms for the relevant pathway (Pathmodel cannot yet automate atom numbering during reactions)
- No mass-spectrometric m/z constraint or goal molecule is available to anchor the incremental inference

## Inputs

- ASP knowledge base .lp file encoding molecules as atom/bond predicates, known reactions, source and goal molecules (e.g., MAA_pwy.lp)
- Observed m/z values from metabolomic profiling, scaled ×10,000 (e.g., mzfiltering(2703000) for m/z 270.3, mzfiltering(3023117) for m/z 302.3117)
- List of absent molecules to exclude from inference (e.g., Z-palythenic acid via absentmolecules predicates)
- Step limit for the incremental clingo solver (e.g., 100)

## Outputs

- pathmodel_output.lp with newreaction, predictatom, predictbond, and reaction predicates describing the inferred structure and reactions
- pathmodel_data_transformations.tsv and pathmodel_incremental_inference.tsv summarizing inferred transformations
- Rendered structure images of predicted molecules (e.g., MAA1 at m/z 270.3 and the MAA2 intermediate) and pathway diagrams via pathmodel_plot/RDKit

## How to apply

Encode the pathway knowledge base as an .lp file (MAA_pwy.lp) containing molecules as atom and bond predicates, known reactions, a source molecule (sedoheptulose-7-phosphate), and goal molecules (MAA1, palythine). Add metabolomic constraints: mzfiltering(2703000) and mzfiltering(3023117), where m/z values are multiplied by 10,000 because clingo does not handle decimals, plus absentmolecules entries (e.g., Z-palythenic acid) so excluded compounds are never used by the inference. Run 'pathmodel infer -i MAA_pwy.lp -o output_folder -s 100', which first executes ReactionSiteExtraction.lp (extracting before/after reaction sites as atom and bond changes) and MZComputation.lp (computing m/z for known molecules and m/z changes across reactions), then drives the incremental clingo solver in PathModel.lp from source toward the goal within the 100-step limit. Inspect pathmodel_output.lp for newreaction, predictatom, predictbond, and reaction predicates, and pathmodel_data_transformations.tsv / pathmodel_incremental_inference.tsv for the inferred transformation chain (MAA backbone through the MAA2 intermediate to MAA1). Run 'pathmodel_plot -i output_folder' to draw the predicted structure with RDKit. Success requires the MZ-filtered prediction to yield a molecule at m/z 270.3 matching the unassigned peak.

## Related tools

- **PathModel** (ASP pipeline whose PathModel.lp performs incremental m/z-constrained inference of new reactions and metabolite structures from source to goal) — https://github.com/pathmodel/pathmodel
- **clingo** (Grounder/solver providing the incremental mode used by PathModel.lp; m/z values are integer-scaled ×10,000 because clingo does not handle decimals) — https://github.com/potassco/clingo
- **clyngor** (Python wrapper around clingo used by the pathmodel wrapper to run the ASP scripts) — https://github.com/Aluriak/clyngor
- **RDKit** (Cheminformatics library used by pathmodel_plot to draw the predicted molecule structures from inferred atoms and bonds) — https://github.com/rdkit/rdkit/
- **networkx** (Graph library (with graphviz and pygraphviz) used to render the inferred pathway diagram with known reactions in green and new reactions in blue) — https://github.com/pathmodel/pathmodel

## Examples

```
pathmodel infer -i MAA_pwy.lp -o output_folder -s 100
```

## Evaluation signals

- The incremental inference reaches the goal molecules (MAA1, palythine) within the maximal step limit of 100
- The m/z-filtered prediction yields a molecule whose computed mass-to-charge ratio equals the scaled observed value, e.g., 270.3 encoded as mzfiltering(2703000)
- pathmodel_output.lp contains the expected predicates (newreaction, predictatom, predictbond, reaction) describing the inferred structure
- pathmodel_data_transformations.tsv and pathmodel_incremental_inference.tsv show the transformation from the MAA backbone through the MAA2 intermediate to MAA1, including the three putative decarboxylation/dehydration reactions
- Molecules declared via absentmolecules (e.g., Z-palythenic acid) never appear in the inferred pathway

## Limitations

- PathModel is a working prototype and scalability is an open issue
- The main limitation is user time to manually encode the starting knowledge base; atom numbering during metabolic reactions cannot yet be fully automated
- Clingo does not handle decimals, so all m/z values must be multiplied by 10,000 before encoding in mzfiltering predicates
- Enzymes for reactions such as porphyra-334 dehydration and decarboxylation are unknown, so inferred steps may not be genomically tractable

## Evidence
