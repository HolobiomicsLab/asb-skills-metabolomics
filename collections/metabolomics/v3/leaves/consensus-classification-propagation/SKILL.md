---
name: consensus-classification-propagation
description: Use when untargeted LC-MS/MS feature-based molecular networking data
  has both GNPS library matches and SIRIUS/CANOPUS in silico class predictions, and
  you need consistent class annotations propagated to entire subnetworks.
when_to_use_negative:
- You have only library matches with no CANOPUS/in silico class predictions — ConCISE
  fuses library and in silico annotations, so a single annotation source defeats the
  consensus design
- You need per-molecule structural formula or identity elucidation rather than ontology-level
  class labels — ConCISE propagates superclass/class/subclass categories, not structures
- GNPS library matches must be used but lack ChemOnt columns and you cannot obtain
  them (e.g. via the Fiehn Labs classyfire Batch workaround) — the ClassyFire pathway
  will fail
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: ConCISE
  role: Performs the consensus classification and annotation propagation across molecular
    networking subnetworks (CLI/GUI/MyBinder)
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Feature-based molecular networking and spectral library matching supplying
    Node_info.tsv and library match ontologies
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: SIRIUS / CANOPUS
  role: In silico metabolite structure prediction producing the canopus_summary.tsv
    class predictions consumed by ConCISE
  repo: https://bio.informatik.uni-jena.de/software/
- name: ClassyFire
  role: Provides the ChemOnt hierarchical ontology (superclass, class, subclass) used
    for consensus annotation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:21bc5ff3d3b9489434d3d7596ba8fb8b80509020e7d71942371ee44b07367199
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zquinlan/conCISE
  tools:
  - CANOPUS
  - ClassyFire
  - ConCISE
  - GNPS
  - Python
  - SIRIUS
  - SIRIUS / CANOPUS
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# consensus-classification-propagation

## Summary

Propagate chemical ontology annotations (ClassyFire ChemOnt superclass/class/subclass or NPClassifier) across molecular networking subnetworks by finding majority-consensus classifications that fuse GNPS spectral library matches with in silico CANOPUS predictions via the ConCISE tool.

## When to use

Use when untargeted LC-MS/MS feature-based molecular networking data has both GNPS library matches and SIRIUS/CANOPUS in silico class predictions, and you need consistent class annotations propagated to entire subnetworks.

## When NOT to use

- You have only library matches with no CANOPUS/in silico class predictions — ConCISE fuses library and in silico annotations, so a single annotation source defeats the consensus design
- You need per-molecule structural formula or identity elucidation rather than ontology-level class labels — ConCISE propagates superclass/class/subclass categories, not structures
- GNPS library matches must be used but lack ChemOnt columns and you cannot obtain them (e.g. via the Fiehn Labs classyfire Batch workaround) — the ClassyFire pathway will fail

## Inputs

- GNPS task ID or GNPS spectral library match (DBResult) file
- CANOPUS class prediction summary file (canopus_summary.tsv) with ClassyFire ChemOnt annotations
- GNPS feature-based molecular networking info file (Node_info.tsv)

## Outputs

- Consensus annotation file with superclass, class, and subclass assignments per ion feature
- True positive rate table per dataset, consensus-threshold condition, and ontology level (e.g. tpr_summary_table.tsv)

## How to apply

Process each dataset with GNPS feature-based molecular networking to produce the Node_info.tsv networking info file and, optionally, a GNPS spectral library match (DBResult) file; run SIRIUS 4 with CANOPUS to generate canopus_summary.tsv with ChemOnt annotations. Execute the ConCISE CLI with a GNPS task ID (or library match file), the canopus_summary file, the Node_info file, and consensus thresholds — superclass 50%, class 70%, subclass 70% by default — plus a True/False flag choosing NPClassifier instead of ClassyFire ontologies. ConCISE finds consensus annotations of putative annotations and propagates them only to subnetworks where the majority of ion features share identical structural classes, exporting a consensus annotation file per dataset. Validate by comparing consensus superclass, class, and subclass assignments against GNPS-reported compound identities and computing true positive rates per ontology level; the paper's criterion is a TPR greater than 95% across all ChemOnt levels (achieved: 98.88% ± 1.62% for library consensus, 96.1% ± 4.96% for in silico consensus).

## Related tools

- **ConCISE** (Performs the consensus classification and annotation propagation across molecular networking subnetworks (CLI/GUI/MyBinder)) — https://github.com/Zquinlan/conCISE
- **GNPS** (Feature-based molecular networking and spectral library matching supplying Node_info.tsv and library match ontologies) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **SIRIUS / CANOPUS** (In silico metabolite structure prediction producing the canopus_summary.tsv class predictions consumed by ConCISE) — https://bio.informatik.uni-jena.de/software/
- **ClassyFire** (Provides the ChemOnt hierarchical ontology (superclass, class, subclass) used for consensus annotation)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- True positive rate of consensus annotations vs GNPS-reported compound identities exceeds 95% at superclass, class, and subclass levels
- Consensus propagation is limited to subnetworks where the majority of ion features share identical structural classes
- Output consensus file contains populated superclass, class, and subclass columns for propagated ion features
- Default threshold settings (50/70/70) reproduce the paper's benchmark when rerun on MSV000082083, MSV000081731, or MSV000083729
- The proportion of reliably annotated ion features expands (paper reports up to 76%) after consensus propagation

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches; workarounds are switching to NPClassifier (NPC argument True) or manually pulling ClassyFire data via the Fiehn Labs classyfire Batch using InChiKeys from the GNPS DBResult file
- Consensus values are threshold-dependent: superclass 50%, class 70%, subclass 70% defaults; changing thresholds changes which annotations propagate
- Mac GUI is deprecated; Mac users must use the CLI or MyBinder interface

## Evidence

- [abstract] First tool to fuse molecular networking, library matching and in silico class prediction: "Herein we present ConCISE (Consensus Classifications of In Silico Elucidations) which is the first tool to fuse molecular networking, spectral library matching and in silico class predictions"
- [abstract] TPR > 95% across ChemOnt levels: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software (superclass, class, subclass)"
- [abstract] Majority-rule propagation within subnetworks: "By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork"
- [readme] Consensus thresholds 50/70/70 with optional NPClassifier: "Superclass percent consensus (optional; default = 50) ... Class percent consensus (optional; default = 70) ... Subclass percent consensus (optional; default = 70)"
- [readme] GNPS ClassyFire supply gap and workarounds: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches."
