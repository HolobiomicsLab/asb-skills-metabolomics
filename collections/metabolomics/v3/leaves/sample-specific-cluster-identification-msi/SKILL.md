---
name: sample-specific-cluster-identification-msi
description: Use when pre-processing a multi-sample MALDI MSI dataset and you need
  to automatically flag samples whose spectra deviate from the multi-sample spectral
  similarity structure, without manual inspection.
when_to_use_negative:
- Single-sample datasets — the outlier detection relies on spectral similarity across
  multiple samples and cannot identify deviation without a cohort
- Analyses where pixel features have not yet been peak-picked/filtered — run the msi_preprocessing_flow
  peak list generation and peak filtering first
- Cases where you only need spatial tissue segmentation per sample (msi_segmentation_flow)
  rather than cross-sample outlier identification
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: msiFlow
  role: Provides the automated workflows implementing UMAP dimension reduction, HDBSCAN
    clustering, and the sample-specific-cluster outlier detection principle
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Executes the outlier detection and segmentation workflows as automated, parallelisable
    pipeline steps
- name: Docker
  role: Packages the workflows in the msiFlow Docker image so the outlier detection
    can be run reproducibly without programming skills
  repo: https://hub.docker.com/r/phispa1812/msiflow_gui
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:37b348800fa2435d59b35dd05615e3516d1e2e1826297c50967931a89512d5a0
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# sample-specific-cluster-identification-msi

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated multi-sample outlier detection for MALDI MSI pre-processing: pixel-wise peak features from all samples are jointly reduced to two dimensions with UMAP, clustered with HDBSCAN, and clusters in which ≥70% of pixels originate from one sample are labelled sample-specific clusters (SSC); samples whose pixels are dominated by SSCs are flagged as outliers.

## When to use

Use when pre-processing a multi-sample MALDI MSI dataset and you need to automatically flag samples whose spectra deviate from the multi-sample spectral similarity structure, without manual inspection.

## When NOT to use

- Single-sample datasets — the outlier detection relies on spectral similarity across multiple samples and cannot identify deviation without a cohort
- Analyses where pixel features have not yet been peak-picked/filtered — run the msi_preprocessing_flow peak list generation and peak filtering first
- Cases where you only need spatial tissue segmentation per sample (msi_segmentation_flow) rather than cross-sample outlier identification

## Inputs

- Pre-processed MALDI MSI peak features (pixel-wise peak lists) for multiple samples, e.g. from msi_preprocessing_flow on imzML/vendor raw data
- Sample identifiers for each pixel
- SSC threshold parameter (default 70% of pixels from a single sample)

## Outputs

- sample_outliers.csv with a sample identifier field, per-cluster sample composition fractions, and a per-sample outlier flag
- Cluster assignments of pixels in the joint UMAP space

## How to apply

Extract pixel-wise peak features for all samples after peak picking/filtering, then jointly project all pixels into a shared two-dimensional UMAP embedding so pixels from different samples become comparable. Cluster the joint UMAP embedding with HDBSCAN, mirroring the density-based dimension-reduction-and-clustering strategy msiFlow uses in its UMAP-based segmentation workflows. For each cluster, compute the per-sample pixel composition and label a cluster as a sample-specific cluster (SSC) if at least 70% of its pixels originate from one sample. Flag any sample whose pixels are dominated by SSC pixels as an outlier, since the approach relies on spectral similarity across multiple samples — deviating samples form their own clusters. Write cluster assignments, per-cluster sample fractions, and per-sample outlier flags to sample_outliers.csv and validate that the flagged set matches samples whose SSC pixel fraction exceeds the 70% threshold.

## Related tools

- **msiFlow** (Provides the automated workflows implementing UMAP dimension reduction, HDBSCAN clustering, and the sample-specific-cluster outlier detection principle) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Executes the outlier detection and segmentation workflows as automated, parallelisable pipeline steps)
- **Docker** (Packages the workflows in the msiFlow Docker image so the outlier detection can be run reproducibly without programming skills) — https://hub.docker.com/r/phispa1812/msiflow_gui

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git && snakemake --cores 1 -s msi_segmentation_flow/Snakefile
```

## Evaluation signals

- sample_outliers.csv exists and contains both a sample identifier field and an outlier flag field
- The set of flagged outlier samples equals exactly those whose pixel fraction in SSC-dominated clusters exceeds the 70% threshold
- Per-cluster sample fractions in the output sum consistently: an SSC has ≥70% of pixels from a single sample
- Cluster assignments correspond to HDBSCAN labels computed on the joint two-dimensional UMAP embedding of pixels from all samples

## Limitations

- Requires the multi-sample joint projection; the workflow memory footprint for MSI pre-processing exceeds 8 GB RAM (other example workflows are processable on a computer with 8 GB RAM)
- The threshold (70% per-cluster sample composition) is a heuristic; borderline samples near the threshold may be misclassified
- Only meaningful for pre-processed, pixel-wise peak features; raw spectra without peak picking/filtering may dilute the spectral-similarity signal

## Evidence
