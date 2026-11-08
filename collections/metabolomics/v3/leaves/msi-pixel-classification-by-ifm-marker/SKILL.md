---
name: msi-pixel-classification-by-ifm-marker
description: Use when co-registered MALDI MSI and immunofluorescence microscopy data
  are available and you need to assign lipids to marker-defined cell populations (e.g.
  Ly6G+ neutrophils in UPEC-infected bladder) via supervised pixel classification.
when_to_use_negative:
- mziFlow does not include lipid annotation or interpretation — do not use this skill
  alone to assign identities to the ranked m/z features (use databases such as LipidMaps
  separately)
- Input data lacks co-registered IFM marker segmentation, so no pixel-level labels
  can be derived
- The goal is unsupervised spatial segmentation or molecular heterogeneity analysis
  — use msi_segmentation_flow or the UMAP-based workflow instead
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: msiFlow
  role: Provides the molecular_signatures_flow Snakemake workflow that bins pixels
    by thresholded IFM segmentation, trains the XGBoost classifier on pixel spectra,
    and outputs feature_importance.csv
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Docker
  role: Containerized execution environment so the classification workflow runs reproducibly
    across platforms
  repo: https://hub.docker.com/r/phispa1812/msiflow_gui
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:8e8ceebdf1ea82fd21ab94350ed18b177de763dad21e2f6e075779a4e9ead69d
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
  - msiFlow
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# msi-pixel-classification-by-ifm-marker

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a tree-based (XGBoost) classifier on pixel-wise MALDI MSI mass spectra using binary labels derived from thresholded IFM marker segmentation (e.g. Ly6G for neutrophils), to identify lipid signatures such as ether-linked PCs that mark specific cell populations in tissue.

## When to use

Use when co-registered MALDI MSI and immunofluorescence microscopy data are available and you need to assign lipids to marker-defined cell populations (e.g. Ly6G+ neutrophils in UPEC-infected bladder) via supervised pixel classification.

## When NOT to use

- mziFlow does not include lipid annotation or interpretation — do not use this skill alone to assign identities to the ranked m/z features (use databases such as LipidMaps separately)
- Input data lacks co-registered IFM marker segmentation, so no pixel-level labels can be derived
- The goal is unsupervised spatial segmentation or molecular heterogeneity analysis — use msi_segmentation_flow or the UMAP-based workflow instead

## Inputs

- Registered, thresholded IFM marker segmentation mask (e.g. segmented/UPEC_12.tif in bin_imgs directory)
- Pre-processed MALDI MSI peak/binned pixel data (imzML-derived peak list from msi_preprocessing_flow)
- Registration output aligning MSI pixels to IFM segmentation
- Binary pixel class labels (marker-positive vs marker-negative)

## Outputs

- feature_importance.csv with importance and Pearson correlation columns per m/z feature
- Ranked list of lipids classifying marker-positive pixels (top feature: PC O-36:4, 768.59 m/z)
- Trained XGBoost classifier over pixel spectra

## How to apply

First register IFM to MALDI MSI using msiFlow's msi_if_registration_flow and threshold the IFM marker (e.g. Ly6G) with if_segmentation_flow to obtain a binary pixel mask (e.g. segmented/UPEC_12.tif in the bin_imgs directory). Bin pixels into marker-positive and marker-negative classes from the thresholded segmentation, then run the molecular_signatures_flow Snakemake workflow, which trains an XGBoost tree-based classifier on the pixel-wise mass spectra using the binary annotations as labels. Rank lipid features by classifier feature importance and compute Pearson correlations between each lipid's intensity and the marker-positive class. Report top-ranked, positively correlated lipids; in the paper's UTI study this yielded ether-linked PCs as the neutrophil signature, with PC O-36:4 (768.59 m/z) at the highest importance, later confirmed by LC-MS/MS and in situ MS/MS to contain esterified arachidonic acid.

## Related tools

- **msiFlow** (Provides the molecular_signatures_flow Snakemake workflow that bins pixels by thresholded IFM segmentation, trains the XGBoost classifier on pixel spectra, and outputs feature_importance.csv) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Docker** (Containerized execution environment so the classification workflow runs reproducibly across platforms) — https://hub.docker.com/r/phispa1812/msiflow_gui

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git && snakemake --cores 8 molecular_signatures_flow
```

## Evaluation signals

- feature_importance.csv exists and contains an importance column and a Pearson correlation column for every m/z feature
- The top-ranked, positively correlated feature corresponds to PC O-36:4 (768.59 m/z) when applied to the UPEC-infected bladder Ly6G dataset
- Top-classifying lipids are predominantly ether-linked PCs and correlate positively with the Ly6G-positive class
- Segmentation mask thresholding yields plausible Ly6G+ pixel fractions consistent with tissue architecture (neutrophils in lamina propria, urothelium) before classification
- Snakemake workflow (molecular_signatures_flow) completes without errors inside the Docker image

## Limitations

- Cell-specificity is limited by the use of consecutive tissue sections and lateral diffusion of lipids from surrounding cells during MSI sample preparation
- The automated IFM-to-MSI registration approach might be less accurate than landmark-based registration or laser ablation marks
- msiFlow does not include an annotation or interpretation step — lipid identification (e.g. esterified arachidonic acid in PC O-36:4) required LC-MS/MS and in situ MS/MS validation outside the workflow

## Evidence
