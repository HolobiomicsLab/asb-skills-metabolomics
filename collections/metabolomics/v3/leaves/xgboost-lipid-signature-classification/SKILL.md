---
name: xgboost-lipid-signature-classification
description: Use when co-registered MALDI MSI and IFM data are available with a marker
  segmentation mask (e.g. Ly6G-positive/-negative pixels), and the goal is to discover
  which lipid m/z features distinguish annotated cell populations via supervised ML
  classification.
when_to_use_negative:
- No co-registered IFM marker segmentation exists to provide class labels — use msi_segmentation_flow
  (dim. reduction and clustering of MSI data) for unsupervised analysis instead.
- The goal is lipid annotation or interpretation rather than candidate discovery —
  the article states msiFlow 'does not include an annotation or interpretation step'.
- Input is already a feature-importance table or statistical comparison between ROI
  groups — use region_group_analysis_flow (statistical analysis between two groups)
  instead of retraining a classifier.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: msiFlow
  role: Provides the molecular_signatures_flow Snakemake workflow that bins pixels
    by IFM segmentation class, trains the XGBoost classifier, and emits feature_importance.csv
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine automating and parallelizing the classification pipeline steps
- name: Docker
  role: Container image packaging msiFlow workflows so the classification runs platform-independently
  repo: https://hub.docker.com/r/phispa1812/msiflow_gui
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:d7d663fcd81e1129ea0dbb3db06cf6ece4970fc02afc0f8b02763cd3d3f8d61c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Immunodynamics-Engel-Lab/msiflow
  repo_url: https://github.com/Immunodynamics-Engel-Lab/msiflow
  tools:
  - Docker
  - Snakemake
  - msiFlow
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# xgboost-lipid-signature-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train an XGBoost tree-based classifier on pixel-wise MALDI MSI mass spectra using thresholded immunofluorescence (IFM) marker segmentations (e.g. Ly6G for neutrophils) as binary labels, to rank lipid m/z features by feature importance and identify cell-population-specific lipid signatures such as the neutrophil ether-linked PC signature (PC O-36:4, 768.59 m/z) in UPEC-infected bladder tissue.

## When to use

Use when co-registered MALDI MSI and IFM data are available with a marker segmentation mask (e.g. Ly6G-positive/-negative pixels), and the goal is to discover which lipid m/z features distinguish annotated cell populations via supervised ML classification.

## When NOT to use

- No co-registered IFM marker segmentation exists to provide class labels — use msi_segmentation_flow (dim. reduction and clustering of MSI data) for unsupervised analysis instead.
- The goal is lipid annotation or interpretation rather than candidate discovery — the article states msiFlow 'does not include an annotation or interpretation step'.
- Input is already a feature-importance table or statistical comparison between ROI groups — use region_group_analysis_flow (statistical analysis between two groups) instead of retraining a classifier.

## Inputs

- Registered MSI peak data pre-processed by msi_preprocessing_flow (imzML-derived peak list)
- Thresholded IFM segmentation mask of the marker of interest (e.g. segmented/UPEC_12.tif in the bin_imgs directory)
- Binary pixel class annotations derived from thresholding-based IFM marker segmentation (if_segmentation_flow)

## Outputs

- feature_importance.csv with an importance and Pearson correlation column for each m/z feature
- Ranked list of lipid features distinguishing the positive class (e.g. ether-linked PCs, top hit PC O-36:4 at 768.59 m/z)
- ML classification of molecular signatures per ROI (molecular_signatures_flow output)

## How to apply

Run msiFlow's molecular_signatures_flow (a Snakemake workflow shipped inside a Docker image) after msi_if_registration_flow and if_segmentation_flow have produced a registered, thresholded marker mask (e.g. segmented/UPEC_12.tif placed in the bin_imgs directory). Bin pixels into two classes (e.g. Ly6G-positive vs Ly6G-negative) from the thresholded IFM segmentation, then train an XGBoost tree-based classifier on the pixel-wise mass spectra with the binary annotations as labels. Rank each m/z lipid feature by classifier feature importance and compute Pearson correlations between each lipid's intensity and the positive class; report top-ranked, positively correlated features. Validate that feature_importance.csv exists, contains an importance and Pearson correlation column per m/z feature, and that the top positively correlated lipid matches the expected signature (e.g. PC O-36:4 at 768.59 m/z for Ly6G+ neutrophils).

## Related tools

- **msiFlow** (Provides the molecular_signatures_flow Snakemake workflow that bins pixels by IFM segmentation class, trains the XGBoost classifier, and emits feature_importance.csv) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine automating and parallelizing the classification pipeline steps)
- **Docker** (Container image packaging msiFlow workflows so the classification runs platform-independently) — https://hub.docker.com/r/phispa1812/msiflow_gui

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git  # then run the molecular_signatures_flow Snakemake workflow via the msiFlow Docker image
```

## Evaluation signals

- feature_importance.csv exists in the molecular_signatures_flow output directory
- feature_importance.csv contains exactly one importance and one Pearson correlation column per m/z feature
- The top-ranked feature by importance is also positively correlated with the Ly6G-positive class
- The top positively correlated lipid corresponds to PC O-36:4 at 768.59 m/z, as reported in the paper
- Classifier separation of Ly6G-positive vs Ly6G-negative pixels is consistent with the paper's finding that several ether-linked PCs carry the highest feature importance

## Limitations

- Cell-specificity is limited by the use of consecutive tissue sections (IFM and MSI from serial sections) and lateral diffusion of lipids from surrounding cells during MSI sample preparation.
- The automated registration of IFM to MSI might be less accurate than landmark-based registration or laser ablation marks, which can propagate pixel misassignment into training labels.
- The workflow identifies candidate molecular patterns only; annotation (e.g. PC O-36:4 with esterified arachidonic acid) required external confirmation by LC-MS/MS and in situ MS/MS.

## Evidence
