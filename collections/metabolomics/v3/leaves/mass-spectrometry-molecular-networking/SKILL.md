---
name: mass-spectrometry-molecular-networking
description: Use when an untargeted tandem MS dataset has been processed with GNPS
  feature-based molecular networking and in silico annotations (e.g., CANOPUS summaries)
  are available, and consensus chemical classifications must be propagated across
  subnetworks.
when_to_use_negative:
- The analysis goal is de novo metabolite identification rather than propagating ClassyFire/NPClassifier
  structural classifications across molecular networking subnetworks
- No molecular networking or in silico (CANOPUS) annotations exist — ConCISE requires
  both networking info and class prediction files as inputs
- The dataset has already been annotated by a tool that fuses library matching and
  in silico consensus classification, making re-propagation redundant
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: ConCISE
  role: Fuses molecular networking, spectral library matching, and in silico class
    predictions to compute consensus annotations for subnetworks via the CLI
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Feature-based molecular networking produces Node_info.tsv and spectral library
    match files consumed as inputs
- name: SIRIUS
  role: Runs with CANOPUS on each dataset to generate canopus_summary.tsv with in
    silico structural annotations
- name: CANOPUS
  role: Provides ChemOnt class predictions that serve as the in silico putative annotations
    for consensus
- name: ClassyFire
  role: Supplies the ChemOnt hierarchical ontology (superclass, class, subclass) used
    for consensus classification
- name: NPClassifier
  role: Alternative ontology selectable via the NPC=True flag when ClassyFire data
    are unavailable for library matches
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:804f9d04026ad652982def3590c0d7c2a8f6d9a0bee280e205ef0c66cd5484c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zquinlan/conCISE
  tools:
  - CANOPUS
  - ClassyFire
  - ConCISE
  - GNPS
  - NPClassifier
  - Python
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mass-spectrometry-molecular-networking

## Summary

Consensus annotation propagation for untargeted LC-MS/MS ion features that fuses GNPS feature-based molecular networking, spectral library matching, and in silico class predictions (SIRIUS 4/CANOPUS) into ClassyFire ChemOnt superclass/class/subclass assignments for entire molecular subnetworks.

## When to use

Use when an untargeted tandem MS dataset has been processed with GNPS feature-based molecular networking and in silico annotations (e.g., CANOPUS summaries) are available, and consensus chemical classifications must be propagated across subnetworks.

## When NOT to use

- The analysis goal is de novo metabolite identification rather than propagating ClassyFire/NPClassifier structural classifications across molecular networking subnetworks
- No molecular networking or in silico (CANOPUS) annotations exist — ConCISE requires both networking info and class prediction files as inputs
- The dataset has already been annotated by a tool that fuses library matching and in silico consensus classification, making re-propagation redundant

## Inputs

- GNPS task ID or GNPS spectral library match file (DBResult with superclass/class/subclass columns)
- CANOPUS class prediction summary file (canopus_summary.tsv from SIRIUS 4)
- GNPS feature-based molecular networking info file (Node_info.tsv)

## Outputs

- Consensus annotation file with superclass, class, and subclass assignments per ion feature
- True positive rate summary per ontology level (e.g., tpr_summary_table.tsv)

## How to apply

Process each dataset with GNPS feature-based molecular networking to produce the networking info file (Node_info.tsv) and, if library matches exist, the GNPS spectral library match file; separately run SIRIUS 4 with CANOPUS to generate canopus_summary.tsv with ClassyFire ChemOnt annotations. Run ConCISE via CLI with consensus thresholds of 50% for superclass and 70% for class and subclass (e.g., 'python3 src/conciseCLI.py <GNPS_task_ID> canopus_summary.tsv Node_info.tsv 50 70 70 False'), propagating an annotation to a subnetwork only when the structural class is identical for the majority of ion features. Set the NPClassifier flag to True to substitute NPClassifier ontologies when ClassyFire data are unavailable for library matches. Evaluate by comparing consensus superclass/class/subclass assignments against GNPS-reported compound identities, computing true positive rates per ontology level (expected >95% at all levels; reported 98.88% ± 1.62% for library and 96.1% ± 4.96% for in silico consensus).

## Related tools

- **ConCISE** (Fuses molecular networking, spectral library matching, and in silico class predictions to compute consensus annotations for subnetworks via the CLI) — https://github.com/Zquinlan/conCISE
- **GNPS** (Feature-based molecular networking produces Node_info.tsv and spectral library match files consumed as inputs)
- **SIRIUS** (Runs with CANOPUS on each dataset to generate canopus_summary.tsv with in silico structural annotations)
- **CANOPUS** (Provides ChemOnt class predictions that serve as the in silico putative annotations for consensus)
- **ClassyFire** (Supplies the ChemOnt hierarchical ontology (superclass, class, subclass) used for consensus classification)
- **NPClassifier** (Alternative ontology selectable via the NPC=True flag when ClassyFire data are unavailable for library matches)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- True positive rate across all ChemOnt levels (superclass, class, subclass) exceeds 95%
- Library-match consensus TPR near 98.88% ± 1.62% and in silico consensus TPR near 96.1% ± 4.96% on the three MassIVE case-study datasets (MSV000082083, MSV000081731, MSV000083729)
- Proportion of reliably annotated ion features expands up to 76% after consensus propagation
- Consensus annotations are only propagated when the structural class is identical for the majority of ion features within a subnetwork
- Consensus thresholds match the defaults/validated settings: 50 (superclass), 70 (class), 70 (subclass)

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches; workarounds are to use NPClassifier (NPC=True) or manually pull ClassyFire data by copying InChiKeys from the GNPS DBResult file into the Fiehn Labs classyFire Batch service and merging columns back
- The Mac GUI is deprecated; Mac users must use the CLI or myBinder interfaces
- Annotation propagation is deliberately limited to classes identical for the majority of ion features in a subnetwork, so sparse or heterogeneous subnetworks may not receive consensus annotations

## Evidence

- [abstract] first tool to fuse molecular networking, spectral library matching and in silico class predictions: "Herein we present ConCISE (Consensus Classifications of In Silico Elucidations) which is the first tool to fuse molecular networking, spectral library matching and in silico class predictions"
- [abstract] true positive rate greater than 95% across all levels: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software (superclass, class, subclass)"
- [methods] consensus annotation of putative annotations using ClassyFire ontologies: "ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS for library spectral matches and in silico putative annotations."
- [readme] CLI arguments and thresholds: "python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False"
- [readme] GNPS ClassyFire availability workaround: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches. There are two workarounds: 1) use NPClassifier instead of ClassyFire"
