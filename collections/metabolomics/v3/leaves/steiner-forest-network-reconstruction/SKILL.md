---
name: steiner-forest-network-reconstruction
description: Use when raw LC-MS spectral peaks have been ranked by deep-learning feature
  attribution (e.g., ensemble RISE contribution scores) and the goal is to link those
  m/z signals to hidden metabolites, proteins, and disease pathways via network propagation.
when_to_use_negative:
- Peaks have not been prioritized by model feature attribution — running PIUMet on
  the full untargeted peak list would propagate noise instead of disease-specific
  signals
- The analysis goal is conventional peak-picking/alignment/annotation-based metabolomics;
  DeepMSProfiler and this downstream network step are designed to bypass peak extraction
  and work directly on raw LC-MS signals
- Only already-identified metabolite names are available (no m/z or MS2 data) — the
  prize-collecting Steiner Forest step requires annotated m/z peaks as prizes
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: PIUMet
  role: Prize-collecting Steiner Forest algorithm that constructs the disease-related
    metabolite-protein network from annotated m/z prizes on the iRefIndex/HMDB/Recon2
    interactome
- name: DeepMSProfiler
  role: Produces the ensemble RISE feature heatmap (ensemble_RISE.npy) whose contribution
    scores define the high-contribution m/z peaks used as network prizes
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: MetaboAnalyst
  role: Joint pathway enrichment of the network metabolites/proteins, filtering enriched
    pathways at FDR < 0.05
- name: Python (>= 3.6)
  role: Runtime environment for DeepMSProfiler feature extraction (TensorFlow 2.2.0
    / Keras 2.3.1) and downstream network handling
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:dfe8f131eb875a742846fb1ce51dd5da9c0dfe729e3582c02ea80e3f24428888
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yjdeng9/DeepMSProfiler
  repo_url: https://github.com/yjdeng9/DeepMSProfiler
  tools:
  - DeepMSProfiler
  - MetaboAnalyst
  - PIUMet
  - Python (>= 3.6)
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# steiner-forest-network-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs disease-related metabolite-protein interaction networks from high-contribution LC-MS m/z peaks using PIUMet's prize-collecting Steiner Forest algorithm, then annotates the resulting subnetwork with pathway enrichment (e.g., LUAD: 82 proteins, 121 metabolites).

## When to use

Use when raw LC-MS spectral peaks have been ranked by deep-learning feature attribution (e.g., ensemble RISE contribution scores) and the goal is to link those m/z signals to hidden metabolites, proteins, and disease pathways via network propagation.

## When NOT to use

- Peaks have not been prioritized by model feature attribution — running PIUMet on the full untargeted peak list would propagate noise instead of disease-specific signals
- The analysis goal is conventional peak-picking/alignment/annotation-based metabolomics; DeepMSProfiler and this downstream network step are designed to bypass peak extraction and work directly on raw LC-MS signals
- Only already-identified metabolite names are available (no m/z or MS2 data) — the prize-collecting Steiner Forest step requires annotated m/z peaks as prizes

## Inputs

- ensemble_RISE.npy feature heatmap from DeepMSProfiler run_feature, with per-peak contribution scores
- Ranked m/z peak list (contribution score > 0.70) with MS2 spectra for annotation
- HMDB v3 metabolite reference annotations
- iRefIndex v13 protein-protein and protein-metabolite interaction network
- Recon2 metabolic model

## Outputs

- luad_metabolite_protein_network.tsv — disease-specific metabolite-protein subnetwork (82 proteins, 121 metabolites for LUAD)
- MetaboAnalyst joint pathway enrichment results (pathways with FDR < 0.05)
- Network and pathway visualization panels (corresponding to Fig. 5g-l)

## How to apply

First load the ensemble RISE feature heatmap (ensemble_RISE.npy) produced by DeepMSProfiler's run_feature step and rank spectral peaks by contribution score, retaining only peaks with contribution score > 0.70. Annotate the retained peaks by matching their m/z values and MS2 spectra against HMDB (v3) metabolite reference annotations. Run PIUMet, the prize-collecting Steiner Forest algorithm, with the annotated metabolites as prizes on the iRefIndex (v13) protein-protein and protein-metabolite interaction network, together with HMDB v3 and Recon2, to extract a disease-specific subnetwork (e.g., the LUAD network of 82 proteins and 121 metabolites saved as luad_metabolite_protein_network.tsv). Finally, submit the network metabolites/proteins to MetaboAnalyst joint pathway enrichment and retain pathways passing FDR < 0.05, visualizing the network and pathways as the Fig. 5g-l panel set.

## Related tools

- **PIUMet** (Prize-collecting Steiner Forest algorithm that constructs the disease-related metabolite-protein network from annotated m/z prizes on the iRefIndex/HMDB/Recon2 interactome)
- **DeepMSProfiler** (Produces the ensemble RISE feature heatmap (ensemble_RISE.npy) whose contribution scores define the high-contribution m/z peaks used as network prizes) — https://github.com/yjdeng9/DeepMSProfiler
- **MetaboAnalyst** (Joint pathway enrichment of the network metabolites/proteins, filtering enriched pathways at FDR < 0.05)
- **Python (>= 3.6)** (Runtime environment for DeepMSProfiler feature extraction (TensorFlow 2.2.0 / Keras 2.3.1) and downstream network handling)

## Examples

```
python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_feature
```

## Evaluation signals

- The reconstructed subnetwork contains exactly 82 proteins and 121 metabolites (LUAD case)
- All reported enriched pathways from MetaboAnalyst satisfy FDR < 0.05
- Only peaks with contribution score > 0.70 from ensemble_RISE.npy were retained as prizes
- Output file luad_metabolite_protein_network.tsv is produced with protein and metabolite columns traceable to iRefIndex v13 / HMDB v3 / Recon2 identifiers

## Limitations

- Pre-trained DeepMSProfiler model access requires contacting the author; without it, the ensemble RISE heatmap must be regenerated by retraining the 10-run DenseNet121 ensemble
- Network reconstruction is bounded by iRefIndex v13, HMDB v3 and Recon2 coverage — metabolites absent from these databases cannot be linked
- The workflow was demonstrated for LUAD serum metabolomics; thresholds (0.70 contribution score) and expected network size are disease/dataset specific

## Evidence

- [other] PIUMet constructs disease-related metabolite-protein networks from input m/z peaks using the prize-collecting Steiner Forest algorithm, integrating iRefIndex (v13), HMDB (v3) and Recon2 databases: [evidence span withheld — anchor status: not_found]
