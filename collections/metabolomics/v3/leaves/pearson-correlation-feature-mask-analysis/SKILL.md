---
name: pearson-correlation-feature-mask-analysis
description: Use when a registered multimodal MSI dataset has a thresholded binary
  pixel annotation (e.g. Ly6G IFM segmentation mask) and the goal is to identify the
  molecular signature — such as ether-linked PCs like PC O-36:4 — of a specific cell
  population.
when_to_use_negative:
- You only have MSI data without a co-registered, thresholded marker segmentation
  — no binary labels exist for class training
- You need definitive lipid annotation or biological interpretation — msiFlow explicitly
  does not include an annotation or interpretation step
- Input is already a feature-importance table — this skill trains the classifier and
  computes correlations itself
tools:
- name: msiFlow
  role: Provides the molecular_signatures_flow Snakemake workflow that trains the
    XGBoost classifier on Ly6G-annotated pixels and outputs feature importance plus
    Pearson correlations
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine automating and parallelizing the molecular signature analysis
    steps
- name: Docker
  role: Containerizes msiFlow so the signature-classification workflows can be run
    reproducibly without local installation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:72feacff180fea17fed5507052e0658af5d8cd45c6edfaa6c55515de14e3174e
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pearson-correlation-feature-mask-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ranks m/z features from co-registered MALDI MSI and IFM data by combining XGBoost classifier feature importance with Pearson correlations of each lipid's intensity against a binary marker-mask class (e.g. Ly6G+ vs Ly6G− pixels), as implemented in msiFlow's molecular_signatures_flow.

## When to use

Use when a registered multimodal MSI dataset has a thresholded binary pixel annotation (e.g. Ly6G IFM segmentation mask) and the goal is to identify the molecular signature — such as ether-linked PCs like PC O-36:4 — of a specific cell population.

## When NOT to use

- You only have MSI data without a co-registered, thresholded marker segmentation — no binary labels exist for class training
- You need definitive lipid annotation or biological interpretation — msiFlow explicitly does not include an annotation or interpretation step
- Input is already a feature-importance table — this skill trains the classifier and computes correlations itself

## Inputs

- Registered/pre-processed MALDI MSI peak data (pixel-wise mass spectra) from msi_preprocessing_flow
- Thresholded IFM segmentation mask for the cell-type marker (e.g. segmented/UPEC_12.tif from if_segmentation_flow)
- Binary pixel class labels (marker-positive vs marker-negative) derived from the mask
- Imaging data from UPEC-infected bladder tissue sections (example data on Zenodo: 10.5281/zenodo.11913042)

## Outputs

- feature_importance.csv with an importance and a Pearson correlation value for each m/z feature
- Ranked list of signature lipids positively correlated with the marker-positive class (top: PC O-36:4, 768.59 m/z)

## How to apply

Run msiFlow's molecular_signatures_flow (a Snakemake workflow, shipped as a Docker image) after the msi_if_registration_flow and if_segmentation_flow steps have produced a registered MSI peak table and a thresholded marker mask (e.g. segmented/UPEC_12.tif). Bin pixels into the positive and negative classes using the thresholded segmentation, then train an XGBoost tree-based classifier on pixel-wise mass spectra using the binary labels. Rank each m/z feature by classifier feature importance and compute the Pearson correlation between each lipid's intensity and the positive class; the top-ranked, positively correlated features form the signature (the paper reports ether-linked PCs, with PC O-36:4 at 768.59 m/z as the top neutrophil lipid, confirmed to carry esterified arachidonic acid). Validate by checking feature_importance.csv contains an importance column and a Pearson correlation column per m/z feature, and that the top positively correlated feature is PC O-36:4 (768.59 m/z). Note that msiFlow itself does not perform lipid annotation — tentative annotations (e.g. via LipidMaps mass matching) or orthogonal confirmation (LC-MS/MS, in situ MS/MS) must be done separately.

## Related tools

- **msiFlow** (Provides the molecular_signatures_flow Snakemake workflow that trains the XGBoost classifier on Ly6G-annotated pixels and outputs feature importance plus Pearson correlations) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine automating and parallelizing the molecular signature analysis steps)
- **Docker** (Containerizes msiFlow so the signature-classification workflows can be run reproducibly without local installation)

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git && snakemake --core molecular_signatures_flow
```

## Evaluation signals

- feature_importance.csv exists and contains both an importance column and a Pearson correlation column for every m/z feature
- The top-ranked positively correlated lipid corresponds to PC O-36:4 at 768.59 m/z
- Top-ranked features are predominantly ether-linked PCs, consistent with the reported neutrophil signature
- Pixel class assignment matches the thresholded Ly6G IFM segmentation mask (Ly6G+ vs Ly6G− binning)
- Top lipids show positive Pearson correlation with the Ly6G-positive class, matching spatial enrichment in neutrophil-rich regions

## Limitations

- msiFlow does not include an annotation or interpretation step — lipid IDs (e.g. PC O-36:4) require external mass matching (LipidMaps) or confirmation by LC-MS/MS and in situ MS/MS
- Cell-specificity is limited because IFM and MSI are from consecutive tissue sections and lipids laterally diffuse during MSI sample preparation
- The automated IFM-to-MSI registration may be less accurate than landmark-based registration or laser ablation marks, propagating error into pixel class labels

## Evidence
