---
name: umap-dimensionality-reduction-of-msi-pixel-spectra
description: Use when raw imzML MALDI MSI pixel spectra need unsupervised grouping
  by spectral similarity — e.g. for tissue segmentation, matrix/off-tissue pixel detection,
  or resolving molecular heterogeneity of a cell population — and no cluster number
  is known a priori.
when_to_use_negative:
- Input is already a reduced/feature table (e.g. peak list or ROI intensity matrix)
  rather than pixel-level spectra — use region_group_analysis_flow or statistical
  workflows instead
- A supervised comparison between sample groups is the goal (use molecular_signatures_flow
  ML classification), since UMAP is an unsupervised exploratory embedding
- The dataset lacks sufficient pixels per sample or memory (the segmentation example
  data requires at least 120 GB RAM), making a full-sample UMAP embedding infeasible
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: msiFlow
  role: End-to-end Snakemake framework (msi_segmentation_flow, molecular_heterogeneity_flow)
    that orchestrates UMAP reduction of MALDI MSI pixel spectra and downstream clustering
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: imzy
  role: Python reader interface for loading raw imzML (and Bruker/Waters) pixel spectra
    before UMAP embedding
  repo: https://github.com/vandeplaslab/imzy
- name: UMAP
  role: Performs the 2D non-linear dimensionality reduction of pixel spectra by spectral
    similarity
- name: HDBSCAN
  role: Density-based clustering applied to the 2D UMAP embedding to recover clusters
    without specifying cluster number a priori
- name: Snakemake
  role: Workflow engine automating and parallelising the UMAP-based segmentation steps
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:f39552fd8e7997bf5976a2a390543519135aff7d698d8fb9cf55890272c5e4f7
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
  - imzy
  - msiFlow
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# umap-dimensionality-reduction-of-msi-pixel-spectra

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduce each sample's MALDI MSI pixel spectra (e.g. t-MALDI-2 imzML data) to a 2-dimensional UMAP embedding that groups pixels by spectral similarity, as implemented in msiFlow's segmentation and molecular-heterogeneity workflows prior to HDBSCAN clustering.

## When to use

Use when raw imzML MALDI MSI pixel spectra need unsupervised grouping by spectral similarity — e.g. for tissue segmentation, matrix/off-tissue pixel detection, or resolving molecular heterogeneity of a cell population — and no cluster number is known a priori.

## When NOT to use

- Input is already a reduced/feature table (e.g. peak list or ROI intensity matrix) rather than pixel-level spectra — use region_group_analysis_flow or statistical workflows instead
- A supervised comparison between sample groups is the goal (use molecular_signatures_flow ML classification), since UMAP is an unsupervised exploratory embedding
- The dataset lacks sufficient pixels per sample or memory (the segmentation example data requires at least 120 GB RAM), making a full-sample UMAP embedding infeasible

## Inputs

- Raw MALDI MSI pixel spectra in imzML format (.imzML/.ibd), read via the imzy reader
- Optionally, a binary IFM segmentation mask (e.g. registered/UPEC_12.tif from msi_if_registration_flow and if_segmentation_flow) to restrict pixels to a cell population
- Demo/example data from Zenodo record 10.5281/zenodo.11913042 or the msiFlow demo dataset (github.com/Immunodynamics-Engel-Lab/msiflow/tree/main/demo)

## Outputs

- 2D UMAP embedding of pixel spectra per sample (pixel coordinates in embedding space)
- Downstream HDBSCAN cluster labels per pixel (e.g. three neutrophil clusters or a border-connected matrix cluster)
- Cluster-to-tissue-composition table (cluster_tissue_composition.csv) or post-processed binary matrix mask (matrix_mask.tif)

## How to apply

Load raw imzML pixel spectra with the imzy reader interface within an msiFlow Snakemake workflow (msi_segmentation_flow for tissue segmentation/matrix detection, or the molecular_heterogeneity_flow for ROI-level analysis). Restrict pixels to the region of interest when needed, e.g. pixels inside a thresholding-segmented Ly6G IFM mask for neutrophil heterogeneity, or all pixels of the measured area for matrix detection. Run UMAP to embed each sample's spectra in 2 dimensions, relying on spectral similarity so that spatially coherent structures (tissue regions, matrix shell) map to separable areas of the embedding. Rationale: the 2D embedding makes subsequent density-based clustering (HDBSCAN) tractable and requires no prior definition of matrix peaks or cluster counts — the paper's matrix detection works whole-sample because it exploits tissue architecture and borders visible in the embedding. Downstream, cluster the UMAP embedding with HDBSCAN and post-process the resulting labels (e.g. border-connectivity for matrix selection, morphological closing for masks).

## Related tools

- **msiFlow** (End-to-end Snakemake framework (msi_segmentation_flow, molecular_heterogeneity_flow) that orchestrates UMAP reduction of MALDI MSI pixel spectra and downstream clustering) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **imzy** (Python reader interface for loading raw imzML (and Bruker/Waters) pixel spectra before UMAP embedding) — https://github.com/vandeplaslab/imzy
- **UMAP** (Performs the 2D non-linear dimensionality reduction of pixel spectra by spectral similarity)
- **HDBSCAN** (Density-based clustering applied to the 2D UMAP embedding to recover clusters without specifying cluster number a priori)
- **Snakemake** (Workflow engine automating and parallelising the UMAP-based segmentation steps)

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git && snakemake --use-conda --cores 1 -s workflow/Snakefile msi_segmentation_flow
```

## Evaluation signals

- The 2D UMAP embedding shows spatially coherent separation between matrix/off-tissue pixels and tissue-covered pixels when coloured by pixel position
- HDBSCAN on the embedding recovers approximately three Ly6G+ neutrophil clusters with the reported localisations: ~83% urothelium, ~88% lamina propria, and a ~53/46% split between lamina propria and muscle
- For matrix detection, the selected cluster is the border-connected cluster containing the most pixels, and the post-processed mask forms a coherent matrix shell around the measured tissue area
- The embedding is produced per sample and is reproducible within the Snakemake workflow, matching the expected output files shipped with the msiFlow demo dataset
- Downstream mask outputs (matrix_mask.tif) exclude tissue-covered pixels and cover the matrix regions surrounding the tissue

## Limitations

- The segmentation example data requires at least 120 GB RAM, so full-sample UMAP may need a smaller demo sample or high-memory infrastructure
- msiFlow does not include an annotation or interpretation step; UMAP clustering identifies interesting molecular patterns and preliminary candidates only
- mRNA/cluster results reflect pixel-level spectra on consecutive tissue sections, so complete cell-specificity is limited by lateral diffusion of lipids during MSI sample preparation

## Evidence

- [results] msiFlow reduces each sample's MALDI-2 MSI data to two dimensions with UMAP followed by HDBSCAN clustering: [evidence span withheld — anchor status: not_found]
- [results] HDBSCAN without a priori cluster number: [evidence span withheld — anchor status: not_found]
