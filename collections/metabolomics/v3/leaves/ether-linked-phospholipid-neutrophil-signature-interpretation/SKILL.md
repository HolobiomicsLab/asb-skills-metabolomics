---
name: ether-linked-phospholipid-neutrophil-signature-interpretation
description: Use when an XGBoost (or similar tree-based) classifier trained on Ly6G-annotated
  pixel spectra from co-registered MALDI MSI/IFM data yields feature_importance.csv,
  and you must identify and biologically interpret the top neutrophil lipids.
when_to_use_negative:
- The aim is lipid annotation or interpretation rather than signature discovery —
  the article states 'So far msiFlow does not include an annotation or interpretation
  step', so confirmatory annotation needs external tools such as LipidMaps mass matching
  or LC-MS/MS/in situ MS/MS
- Input is raw unregistered MALDI MSI or IFM data that has not passed msi_preprocessing_flow
  and msi_if_registration_flow, so no Ly6G-annotated pixel spectra exist yet
- The tissue or cell type of interest is not neutrophil-related — the ether-linked
  PC signature is specific to Ly6G+ neutrophils in UPEC-infected bladder and PC O-36:4
  is not detectable at the urothelium infection site
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: msiFlow
  role: Provides molecular_signatures_flow (Snakemake workflow) that trains the XGBoost
    classifier on Ly6G-annotated pixel spectra and outputs feature_importance.csv
    with importance and Pearson correlation columns
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Docker
  role: Packages msiFlow workflows into a Docker image so the signature-classification
    run is reproducible without programming setup
- name: Snakemake
  role: Automates and parallelises the workflow steps (pre-processing, registration,
    segmentation, molecular signatures) that produce the classifier inputs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:7133a52d4c89cead721e73d3dc7815009386c52d3fcc098c04e8971e388f2486
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
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ether-linked-phospholipid-neutrophil-signature-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets ML-derived molecular signatures from multimodal MALDI MSI/IFM data as a neutrophil lipidomic signature, centred on ether-linked PCs (PC O-36:4 at 768.59 m/z) carrying esterified arachidonic acid in UPEC-infected bladder tissue.

## When to use

Use when an XGBoost (or similar tree-based) classifier trained on Ly6G-annotated pixel spectra from co-registered MALDI MSI/IFM data yields feature_importance.csv, and you must identify and biologically interpret the top neutrophil lipids.

## When NOT to use

- The aim is lipid annotation or interpretation rather than signature discovery — the article states 'So far msiFlow does not include an annotation or interpretation step', so confirmatory annotation needs external tools such as LipidMaps mass matching or LC-MS/MS/in situ MS/MS
- Input is raw unregistered MALDI MSI or IFM data that has not passed msi_preprocessing_flow and msi_if_registration_flow, so no Ly6G-annotated pixel spectra exist yet
- The tissue or cell type of interest is not neutrophil-related — the ether-linked PC signature is specific to Ly6G+ neutrophils in UPEC-infected bladder and PC O-36:4 is not detectable at the urothelium infection site

## Inputs

- feature_importance.csv from msiFlow's molecular_signatures_flow (importance and Pearson correlation columns per m/z feature)
- Registered and thresholded Ly6G IFM segmentation mask (e.g. segmented/UPEC_12.tif in the bin_imgs directory)
- Pre-processed MALDI MSI peak data (peak list from msi_preprocessing_flow)
- Trained XGBoost classifier with binary Ly6G pixel labels

## Outputs

- Ranked list of lipids by classifier feature importance with Pearson correlations to the Ly6G-positive class
- Neutrophil lipidomic signature of ether-linked PCs, with PC O-36:4 (768.59 m/z) as the top feature
- Biological interpretation: esterified arachidonic acid in ether-linked PCs marking neutrophil spatial heterogeneity in infected tissue

## How to apply

After msiFlow's molecular_signatures_flow trains an XGBoost classifier on pixel-wise mass spectra binned into Ly6G-positive and Ly6G-negative classes using the thresholded IFM segmentation mask (e.g. segmented/UPEC_12.tif), rank m/z features by classifier feature importance and compute Pearson correlations with the Ly6G-positive class. Validate that feature_importance.csv contains an importance and Pearson correlation column for each m/z feature, and that the top-ranked, positively correlated lipid corresponds to PC O-36:4 (768.59 m/z). Interpret the resulting ether-linked PC signature biologically: the paper reports PC O-36:4 as most abundant in neutrophils after extravasation in the lamina propria but not at the infection site in the urothelium, and LC-MS/MS plus in situ MS/MS confirmed esterified arachidonic acid. Read-out: spatial regulation of ether-linked phospholipids containing arachidonic acid reveals the molecular heterogeneity of neutrophils during the immune response.

## Related tools

- **msiFlow** (Provides molecular_signatures_flow (Snakemake workflow) that trains the XGBoost classifier on Ly6G-annotated pixel spectra and outputs feature_importance.csv with importance and Pearson correlation columns) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Docker** (Packages msiFlow workflows into a Docker image so the signature-classification run is reproducible without programming setup)
- **Snakemake** (Automates and parallelises the workflow steps (pre-processing, registration, segmentation, molecular signatures) that produce the classifier inputs)

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git
```

## Evaluation signals

- feature_importance.csv exists and contains an importance column and a Pearson correlation column for each m/z feature
- The top-ranked, positively correlated lipid corresponds to PC O-36:4 at 768.59 m/z
- The top lipids are ether-linked PCs, consistent with the reported neutrophil signature
- PC O-36:4 signal localises to Ly6G+ regions in the lamina propria (post-extravasation neutrophils) and is absent from the urothelium infection site
- LC-MS/MS and in situ MS/MS confirm the top lipid contains esterified arachidonic acid

## Limitations

- Cell-specificity is limited because registration uses consecutive tissue sections and lipids laterally diffuse from surrounding cells during MSI sample preparation
- msiFlow itself does not include an annotation or interpretation step; confirming lipid identities (e.g. via LipidMaps mass matching or LC-MS/MS) is outside the workflow
- The automated IFM-to-MSI registration may be less accurate than landmark-based registration or laser ablation marks

## Evidence

- [discussion] top lipid finding: "we applied msiFlow to lipidomic MSI and IFM data of UPEC-infected bladders. This revealed high abundance of ether-linked PCs in neutrophils"
- [discussion] spatial pattern of PC O-36:4: "we found that PC O-36:4 was most abundant in neutrophils after extravasation in the lamina propria, but not detectable at the site of infection in the urothelium"
