---
name: hdbscan-unsupervised-clustering-of-spectral-data
description: Use when you have pixel spectra from registered MALDI MSI data (optionally
  masked to a cell-type region such as Ly6G+ neutrophils) and need to discover the
  number of molecularly distinct clusters without pre-specifying k.
when_to_use_negative:
- You need an annotation or interpretation step — the article states msiFlow 'does
  not include an annotation or interpretation step'; it only identifies interesting
  molecular patterns and preliminary candidates
- Data are already a curated feature/cluster table with a known, fixed number of groups
  — use a supervised or fixed-k method (e.g. the ML classification in molecular_signatures_flow)
  instead
- Input is raw vendor data not yet peak-picked or registered — run msi_preprocessing_flow
  and msi_if_registration_flow first
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: msiFlow
  role: Open-source framework providing the Snakemake workflows (molecular_heterogeneity_flow,
    msi_segmentation_flow) that implement UMAP dimensionality reduction and HDBSCAN
    clustering of MSI spectra end-to-end
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Automates and parallelises the UMAP+HDBSCAN clustering workflows for high-throughput
    processing
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Docker
  role: Docker image packaging msiFlow workflows so clustering can be run reproducibly
    and platform-independently
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:d0d35b07e02c2294a5a9b8bee63d5115b6d40138eb102b0f66882a9bf8856cbf
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
  - HDBSCAN
  - Snakemake
  - msiFlow
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# hdbscan-unsupervised-clustering-of-spectral-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Unsupervised clustering of mass spectrometry imaging (and other spectral) pixel data using UMAP dimensionality reduction followed by HDBSCAN, which resolves molecularly distinct pixel clusters without specifying the cluster number a priori. In msiFlow this powers both automated matrix/outlier detection and the molecular_heterogeneity_flow analysis of cell-type-specific heterogeneity.

## When to use

Use when you have pixel spectra from registered MALDI MSI data (optionally masked to a cell-type region such as Ly6G+ neutrophils) and need to discover the number of molecularly distinct clusters without pre-specifying k.

## When NOT to use

- You need an annotation or interpretation step — the article states msiFlow 'does not include an annotation or interpretation step'; it only identifies interesting molecular patterns and preliminary candidates
- Data are already a curated feature/cluster table with a known, fixed number of groups — use a supervised or fixed-k method (e.g. the ML classification in molecular_signatures_flow) instead
- Input is raw vendor data not yet peak-picked or registered — run msi_preprocessing_flow and msi_if_registration_flow first

## Inputs

- Registered MALDI MSI pixel spectra (e.g. registered/msi_if_registration/registered/UPEC_12.tif)
- Thresholding-segmented IFM cell-type mask (e.g. if_segmentation/segmented/UPEC_12.tif)
- Raw MALDI MSI / IFM image data as processed by msi_preprocessing_flow and msi_if_registration_flow

## Outputs

- HDBSCAN cluster labels per pixel over the UMAP embedding
- Cluster-to-tissue-composition table (cluster_tissue_composition.csv) with percentage composition per tissue compartment
- UMAP embedding and segmentation/clustering visual outputs from molecular_heterogeneity_flow

## How to apply

Load the registered MSI data (e.g. registered/UPEC_12.tif) together with the thresholding-segmented IFM mask (e.g. if_segmentation/segmented/UPEC_12.tif) and extract pixel spectra inside the mask. Apply UMAP dimensionality reduction to the pixel spectra, then cluster the UMAP embedding with HDBSCAN so the number of clusters emerges from data density rather than being fixed in advance. In msiFlow the same UMAP+HDBSCAN pairing is also used for automated matrix pixel detection, which relies on tissue architecture and borders, and for outlier detection based on spectral similarity across multiple samples. Finally, assign each resulting cluster to tissue compartments (urothelium, lamina propria, muscle) and compute the percentage composition per cluster per compartment, saved as cluster_tissue_composition.csv. All steps run as automated Snakemake workflows inside the msiFlow Docker image.

## Related tools

- **msiFlow** (Open-source framework providing the Snakemake workflows (molecular_heterogeneity_flow, msi_segmentation_flow) that implement UMAP dimensionality reduction and HDBSCAN clustering of MSI spectra end-to-end) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Automates and parallelises the UMAP+HDBSCAN clustering workflows for high-throughput processing) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Docker** (Docker image packaging msiFlow workflows so clustering can be run reproducibly and platform-independently) — https://github.com/Immunodynamics-Engel-Lab/msiflow

## Examples

```
snakemake --use-conda --cores 1 --snakefile molecular_heterogeneity_flow/Snakefile
```

## Evaluation signals

- Recovery of approximately three clusters when applied to Ly6G+ pixel spectra from the demo dataset
- Cluster-to-tissue composition matches reported localisations: ~83% urothelium, ~88% lamina propria, and a ~53/46% lamina propria/muscle split
- cluster_tissue_composition.csv is produced with percentage compositions per tissue compartment summing appropriately per cluster
- No cluster number was specified a priori — HDBSCAN density estimates determine k
- For matrix-detection use, detected pixels align with tissue architecture and tissue borders

## Limitations

- msiFlow does not include an annotation or interpretation step, so cluster identities require downstream lipid annotation (e.g. mass matching to LipidMaps) by the user
- Cell-specificity is limited by use of consecutive tissue sections and lateral diffusion of lipids from surrounding cells during MSI sample preparation, so clusters may not be purely cell-type-specific
- HDBSCAN cluster counts and compositions can vary with UMAP parameters and dataset density; validation against reported localisations is recommended

## Evidence

- [other] HDBSCAN clusters without specifying cluster number: [evidence span withheld — anchor status: not_found]
