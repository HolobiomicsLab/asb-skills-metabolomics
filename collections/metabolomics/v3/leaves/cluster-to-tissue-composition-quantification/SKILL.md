---
name: cluster-to-tissue-composition-quantification
description: Use when you have pixel-level MSI clusters (e.g. HDBSCAN clusters within
  a cell-type mask such as Ly6G+ neutrophils) and registered/segmented tissue-region
  annotations, and need to quantify the percentage composition of each cluster per
  tissue compartment.
when_to_use_negative:
- Input is already a per-cluster region composition table or ROI-level statistics
  — no clustering/assignment is needed
- Tissue regions have not been segmented or registered to the MSI pixel grid (if_segmentation_flow
  / msi_if_registration_flow outputs are missing)
- The goal is lipid annotation or interpretation rather than spatial localisation
  quantification — msiFlow explicitly does not include an annotation step
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: msiFlow
  role: End-to-end workflow framework providing the molecular_heterogeneity_flow (UMAP-based
    heterogeneity analysis) and if_segmentation_flow used to obtain clusters and region
    masks
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Automates and parallelises the workflow steps from registration through segmentation
    to the cluster-tissue composition analysis
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: UMAP
  role: Dimensionality reduction of Ly6G+ pixel spectra prior to clustering in the
    molecular heterogeneity workflow
- name: HDBSCAN
  role: Density-based clustering of the UMAP embedding to identify molecularly distinct
    pixel clusters without specifying cluster number a priori
- name: Docker
  role: Container image packaging the msiFlow workflows for platform-independent execution
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:618ce7612e0bff0f2ee493e53511df84a5ca499ffd85b37a678d604966ce124e
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
  - UMAP
  - msiFlow
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cluster-to-tissue-composition-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how each molecularly distinct pixel cluster from unsupervised clustering (e.g. UMAP+HDBSCAN) of MSI spectra localises to segmented tissue regions (urothelium, lamina propria, muscle), producing a cluster-to-tissue-percentage composition table.

## When to use

Use when you have pixel-level MSI clusters (e.g. HDBSCAN clusters within a cell-type mask such as Ly6G+ neutrophils) and registered/segmented tissue-region annotations, and need to quantify the percentage composition of each cluster per tissue compartment.

## When NOT to use

- Input is already a per-cluster region composition table or ROI-level statistics — no clustering/assignment is needed
- Tissue regions have not been segmented or registered to the MSI pixel grid (if_segmentation_flow / msi_if_registration_flow outputs are missing)
- The goal is lipid annotation or interpretation rather than spatial localisation quantification — msiFlow explicitly does not include an annotation step

## Inputs

- Registered MALDI MSI data (msi_if_registration_flow output, e.g. registered/UPEC_12.tif)
- Thresholding-segmented IFM marker mask (if_segmentation_flow output, e.g. segmented/UPEC_12.tif)
- UMAP embedding and HDBSCAN cluster labels of Ly6G+ pixel spectra

## Outputs

- cluster_tissue_composition.csv: per-cluster percentage composition across tissue regions (urothelium, lamina propria, muscle)

## How to apply

Load the registered MALDI MSI data and the thresholding-segmented IFM mask (e.g. if_segmentation output UPEC_12.tif) from the msiFlow demo chain, and extract pixel spectra falling inside the Ly6G-positive regions. Reduce the spectra with UMAP and cluster the embedding with HDBSCAN (cluster number not fixed a priori), as implemented in the msiFlow molecular_heterogeneity workflow. Assign each resulting cluster to the tissue regions (urothelium, lamina propria, muscle) and compute the percentage composition of each cluster per compartment. Write the resulting table as cluster_tissue_composition.csv. Correct application is judged by recovering approximately three clusters with the reported localisation pattern (~83% urothelium, ~88% lamina propria, and a ~53/46% split between lamina propria and muscle).

## Related tools

- **msiFlow** (End-to-end workflow framework providing the molecular_heterogeneity_flow (UMAP-based heterogeneity analysis) and if_segmentation_flow used to obtain clusters and region masks) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Automates and parallelises the workflow steps from registration through segmentation to the cluster-tissue composition analysis) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **UMAP** (Dimensionality reduction of Ly6G+ pixel spectra prior to clustering in the molecular heterogeneity workflow)
- **HDBSCAN** (Density-based clustering of the UMAP embedding to identify molecularly distinct pixel clusters without specifying cluster number a priori)
- **Docker** (Container image packaging the msiFlow workflows for platform-independent execution)

## Examples

```
snakemake --use-conda --cores 1 -s molecular_heterogeneity_flow --config sample=UPEC_12 mask=if_segmentation/segmented/UPEC_12.tif output=cluster_tissue_composition.csv
```

## Evaluation signals

- HDBSCAN recovers approximately three neutrophil clusters from the Ly6G+ pixel spectra
- Cluster 1 is assigned mainly to the lamina propria (~88%)
- Cluster 2 is split between lamina propria (~52.7%) and muscle (~46.4%)
- A third cluster localises predominantly to the urothelium (~83%)
- Output file cluster_tissue_composition.csv exists and row percentages per cluster sum to ~100% across the three tissue regions

## Limitations

- Cell-specificity is incomplete because IFM and MSI are acquired on consecutive tissue sections and lipids can laterally diffuse during MSI sample preparation
- The automated registration approach may be less accurate than landmark-based registration or laser ablation marks, affecting cluster-to-region assignment
- msiFlow does not include an annotation or interpretation step; it identifies molecular patterns and preliminary candidates only

## Evidence

- [results] HDBSCAN clustering of Ly6G+ pixel spectra yielded three neutrophil clusters with distinct tissue localisation: [evidence span withheld — anchor status: not_found]
- [results] cluster 1 is mainly in the lamina propria (88%), while cluster 2 is split between the lamina propria (52.7%) and muscle (46.4%): [evidence span withheld — anchor status: not_found]
- [readme] thresholding-based segmentation of IFM markers | `if_segmentation_flow`: "thresholding-based segmentation of IFM markers | `if_segmentation_flow`"
- [readme] molecular heterogeneity in ROIs by UMAP-based: "molecular heterogeneity in ROIs by UMAP-based"
- [discussion] we acknowledge limitations in achieving complete cell-specificity due to the usage of consecutive tissue sections and lateral diffusion of lipids from surrounding cells during MSI sample preparation.: "we acknowledge limitations in achieving complete cell-specificity due to the usage of consecutive tissue sections and lateral diffusion of lipids from surrounding cells during MSI sample preparation."
