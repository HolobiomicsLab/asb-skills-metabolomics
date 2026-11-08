---
name: umap-joint-embedding-of-multi-sample-pixels
description: Use when pre-processing a multi-sample MALDI MSI dataset and you need
  to identify sample outliers automatically by comparing pixel-level spectral similarity
  across all samples in one embedding.
when_to_use_negative:
- Dataset contains only a single sample — the SSC outlier logic requires spectral
  similarity across multiple samples to be meaningful
- Input is already a per-sample summary feature table rather than pixel-wise peak
  features — the embedding and clustering must operate at pixel level
- The goal is sample clustering or tissue segmentation within one sample rather than
  cross-sample outlier QC
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: msiFlow
  role: Provides the automated Snakemake workflow implementing joint UMAP dimensionality
    reduction plus HDBSCAN clustering and the sample-specific-cluster outlier detection
    for multi-sample MALDI MSI pre-processing
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Orchestrates the UMAP+HDBSCAN outlier-detection workflow steps with parallel
    data processing for high-throughput analyses
- name: Docker
  role: Container image packaging msiFlow's workflows so the outlier detection runs
    reproducibly on any platform
  repo: https://hub.docker.com/r/phispa1812/msiflow_gui
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:13aedd1554178149a2d853bf7c5d2f7a5145a53f68a5b53ea2743a493e34472b
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# umap-joint-embedding-of-multi-sample-pixels

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Jointly project pixel-wise peak features from all samples in a multi-sample MALDI MSI dataset into a shared two-dimensional UMAP embedding so that pixels from different samples become comparable for downstream density-based outlier detection (msiFlow's sample-specific cluster approach).

## When to use

Use when pre-processing a multi-sample MALDI MSI dataset and you need to identify sample outliers automatically by comparing pixel-level spectral similarity across all samples in one embedding.

## When NOT to use

- Dataset contains only a single sample — the SSC outlier logic requires spectral similarity across multiple samples to be meaningful
- Input is already a per-sample summary feature table rather than pixel-wise peak features — the embedding and clustering must operate at pixel level
- The goal is sample clustering or tissue segmentation within one sample rather than cross-sample outlier QC

## Inputs

- Multi-sample MALDI MSI dataset (e.g. imzML or vendor format) with pixel-wise peak features per sample
- Example multi-sample data from Zenodo DOI 10.5281/zenodo.11913042 (processable on 8 GB RAM)
- m/z peak feature table per pixel with sample identifiers

## Outputs

- sample_outliers.csv with a sample identifier field, cluster assignments, per-cluster sample fractions, and per-sample outlier flags
- Two-dimensional joint UMAP embedding of pixels from all samples
- HDBSCAN cluster labels per pixel with sample composition per cluster

## How to apply

Extract pixel-wise peak features for all samples, then reduce the pooled data of all samples jointly to two dimensions with UMAP so pixels from different samples share one comparable latent space (rather than embedding each sample separately). Cluster the joint UMAP embedding with HDBSCAN, the same density-based strategy msiFlow uses in its segmentation workflow ('dim. reduction and clustering of MSI data'). Compute per-cluster sample composition and label any cluster in which >=70% of pixels originate from one sample as a sample-specific cluster (SSC). Flag samples whose pixels are predominantly in SSC clusters as sample outliers; this works because the outlier detection relies on spectral similarity across multiple samples. Write cluster assignments, per-cluster sample fractions, and per-sample outlier flags (e.g. to sample_outliers.csv) for the multi-sample pre-processing QC.

## Related tools

- **msiFlow** (Provides the automated Snakemake workflow implementing joint UMAP dimensionality reduction plus HDBSCAN clustering and the sample-specific-cluster outlier detection for multi-sample MALDI MSI pre-processing) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Orchestrates the UMAP+HDBSCAN outlier-detection workflow steps with parallel data processing for high-throughput analyses)
- **Docker** (Container image packaging msiFlow's workflows so the outlier detection runs reproducibly on any platform) — https://hub.docker.com/r/phispa1812/msiflow_gui

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git && snakemake --use-conda --cores 1 -s msi_preprocessing_flow
```

## Evaluation signals

- sample_outliers.csv exists and contains a sample identifier field and an outlier flag field
- The set of flagged outlier samples equals exactly those whose pixel fraction in SSC-dominated clusters exceeds the 70% threshold
- Per-cluster sample fractions are reported and each SSC satisfies the >=70% single-sample pixel composition rule
- The joint UMAP embedding contains pixels from all samples in one shared two-dimensional space, not per-sample embeddings
- Re-running the workflow reproduces consistent cluster assignments and outlier flags on the same input data

## Limitations

- The msi_preprocessing_flow is more resource-intensive than other workflows — Zenodo example data is stated to be processable on 8 GB RAM only for workflows other than MSI pre-processing and segmentation
- The approach depends on joint spectral similarity across samples, so heterogeneous but valid biological samples may be spuriously flagged
- UMAP and HDBSCAN outputs are stochastic/density-dependent; results can vary with hyperparameters and random state

## Evidence

- [methods] Segmentation workflow uses UMAP + HDBSCAN (dim. reduction and clustering): [evidence span withheld — anchor status: not_found]
