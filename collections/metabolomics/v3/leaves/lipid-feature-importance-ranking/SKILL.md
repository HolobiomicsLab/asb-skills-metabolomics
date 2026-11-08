---
name: lipid-feature-importance-ranking
description: Use when you have MALDI MSI peak data co-registered with an IFM segmentation
  mask (e.g. thresholded Ly6G marker) and need to rank which lipids (m/z features)
  classify marker-positive vs marker-negative pixels.
when_to_use_negative:
- Your goal is lipid annotation/interpretation rather than candidate discovery — msiFlow
  explicitly does not include an annotation or interpretation step (use databases
  such as LipidMaps separately)
- Your labels come from manual ROI drawing rather than a thresholded IFM marker segmentation,
  so the binary pixel-binning premise (marker-positive vs marker-negative classes)
  does not apply
- Your data are single-modality MSI without co-registered microscopy, leaving no IFM-derived
  class labels to train on
edam_operation: http://edamontology.org/operation_2990
tools:
- name: msiFlow
  role: Provides the molecular_signatures_flow Snakemake workflow that bins pixels
    by the IFM segmentation, trains the XGBoost classifier, and emits feature_importance.csv
    with importance and Pearson correlation columns
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Docker
  role: Packages all msiFlow workflows (including molecular_signatures_flow) into
    a Docker image so the classifier training runs reproducibly without local dependency
    setup
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:6f85352278470c8a371464befe499bd9eea9c6018f4199abff46d2b15c7a7dad
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# lipid-feature-importance-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies the spatial lipidomic signature of an image-derived cell population (e.g. Ly6G+ neutrophils) by training a tree-based XGBoost classifier on co-registered MALDI MSI pixel spectra and ranking lipid features by classifier feature importance and Pearson correlation with the marker-positive class, as implemented in msiFlow's molecular_signatures_flow.

## When to use

Use when you have MALDI MSI peak data co-registered with an IFM segmentation mask (e.g. thresholded Ly6G marker) and need to rank which lipids (m/z features) classify marker-positive vs marker-negative pixels.

## When NOT to use

- Your goal is lipid annotation/interpretation rather than candidate discovery — msiFlow explicitly does not include an annotation or interpretation step (use databases such as LipidMaps separately)
- Your labels come from manual ROI drawing rather than a thresholded IFM marker segmentation, so the binary pixel-binning premise (marker-positive vs marker-negative classes) does not apply
- Your data are single-modality MSI without co-registered microscopy, leaving no IFM-derived class labels to train on

## Inputs

- Registered and thresholded IFM segmentation mask (e.g. segmented/UPEC_12.tif in the bin_imgs directory) defining Ly6G-positive vs Ly6G-negative pixels
- Pre-processed MALDI MSI peak data (peak list from msi_preprocessing_flow) co-registered to the microscopy modality
- Binary pixel class labels derived from thresholding-based IFM marker segmentation (if_segmentation_flow)

## Outputs

- feature_importance.csv containing an importance score and a Pearson correlation column for each m/z feature
- Ranked list of top, positively correlated lipids (paper reports ether-linked PCs; top feature PC O-36:4 at 768.59 m/z)
- Trained XGBoost classifier on Ly6G-annotated pixel spectra

## How to apply

Load the registered, thresholded IFM segmentation mask (e.g. segmented/UPEC_12.tif placed in the bin_imgs directory) together with pre-processed MALDI MSI peak data, then bin pixels into marker-positive and marker-negative classes based on the thresholded marker segmentation. Run msiFlow's molecular_signatures_flow (a Snakemake workflow, executed via the provided Docker image) to train an XGBoost tree-based classifier on the pixel-wise mass spectra using the binary Ly6G annotations as labels. Rank lipid features by classifier feature importance and compute a Pearson correlation between each lipid's intensity and the marker-positive class to retain positively correlated candidates. In the UPEC-infected bladder proof-of-concept this surfaced ether-linked PCs as the neutrophil signature, with PC O-36:4 (768.59 m/z, esterified arachidonic acid) as the top-ranked feature. Rationale: coupling ML feature importance with spatial correlation prioritizes lipids that are both discriminative and spatially enriched in the cell type of interest.

## Related tools

- **msiFlow** (Provides the molecular_signatures_flow Snakemake workflow that bins pixels by the IFM segmentation, trains the XGBoost classifier, and emits feature_importance.csv with importance and Pearson correlation columns) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Docker** (Packages all msiFlow workflows (including molecular_signatures_flow) into a Docker image so the classifier training runs reproducibly without local dependency setup)

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git
```

## Evaluation signals

- feature_importance.csv exists and contains an importance column and a Pearson correlation column for every m/z feature
- The top-ranked features are positively correlated with the Ly6G-positive class (positive Pearson correlation, not merely high importance)
- On the UPEC-infected bladder dataset, the top-ranked, positively correlated lipid corresponds to PC O-36:4 at 768.59 m/z
- Pixel counts of Ly6G-positive and Ly6G-negative bins are non-degenerate (both classes populated after thresholding-based segmentation)
- Downstream validation: LC-MS/MS and in situ MS/MS confirm the esterified fatty acid content of the top candidate (arachidonic acid in PC O-36:4)

## Limitations

- Cell-specificity is incomplete because registration uses consecutive tissue sections and lipids laterally diffuse from surrounding cells during MSI sample preparation
- The automated IFM-to-MSI registration approach may be less accurate than landmark-based registration or laser ablation marks
- Tentative lipid annotations require external mass matching to databases such as LipidMaps; msiFlow itself does not annotate
- Top candidates reflect spatial enrichment, not cell-type-exclusive synthesis — e.g. PC O-36:4 was abundant in neutrophils in the lamina propria but not detectable at the urothelium infection site

## Evidence
