---
name: multi-sample-outlier-detection-via-spectral-similarity
description: Use when pre-processing a multi-sample MALDI MSI dataset and you need
  to automatically detect samples whose spectra deviate from the spectral similarity
  expected across samples before downstream analysis.
when_to_use_negative:
- 'Single-sample datasets: the method relies on spectral similarity across multiple
  samples, so with one sample there is no cross-sample reference to define sample-specific
  clusters.'
- Datasets where a manual or landmark-based QC approach is already available and preferred
  over automated outlier flagging.
- Non-MSI imaging data (e.g. IFM) without pixel-wise spectral peak features, since
  the embedding requires peak features comparable across samples.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: msiFlow
  role: Provides the automated workflow implementing the UMAP + HDBSCAN outlier detection
    principle for multi-sample MALDI MSI pre-processing
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Automates and parallelizes the outlier-detection workflow steps for high-throughput
    multi-sample analyses
- name: Docker
  role: Packages msiFlow workflows (including pre-processing with outlier detection)
    into a Docker image for platform-independent reuse
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:5d2bcb6d6efde3aef8bb3450f3e91b34b888ba69143b6edc58189e866b51fc5f
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# multi-sample-outlier-detection-via-spectral-similarity

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated identification of outlier samples in multi-sample MALDI MSI datasets by jointly embedding all pixels from every sample in a shared 2D UMAP space, clustering with HDBSCAN, and flagging samples whose pixels are dominated by sample-specific clusters (SSC). This is the outlier-detection principle used in msiFlow's pre-processing workflows.

## When to use

Use when pre-processing a multi-sample MALDI MSI dataset and you need to automatically detect samples whose spectra deviate from the spectral similarity expected across samples before downstream analysis.

## When NOT to use

- Single-sample datasets: the method relies on spectral similarity across multiple samples, so with one sample there is no cross-sample reference to define sample-specific clusters.
- Datasets where a manual or landmark-based QC approach is already available and preferred over automated outlier flagging.
- Non-MSI imaging data (e.g. IFM) without pixel-wise spectral peak features, since the embedding requires peak features comparable across samples.

## Inputs

- Raw MALDI MSI data for multiple samples (e.g. imzML), obtainable from the example data at Zenodo DOI 10.5281/zenodo.11913042
- Pixel-wise peak features extracted per sample for joint embedding

## Outputs

- sample_outliers.csv containing a sample identifier field, per-cluster sample fractions, and a per-sample outlier flag field
- Per-pixel cluster assignments from HDBSCAN on the joint UMAP embedding

## How to apply

Extract pixel-wise peak features for all samples and jointly project them into a shared low-dimensional (2D) UMAP embedding so pixels from different samples are directly comparable. Cluster the joint UMAP embedding with HDBSCAN, the same density-based clustering strategy msiFlow uses for its UMAP-based segmentation workflows (dim. reduction and clustering of MSI data). For each cluster, compute the per-sample composition and label any cluster in which >=70% of pixels originate from a single sample as a sample-specific cluster (SSC). Flag as outliers any samples whose pixels are dominated by SSC clusters, since msiFlow's outlier detection relies on spectral similarity across multiple samples: in non-outlier samples, pixels mix across samples in the joint embedding. Write cluster assignments, per-cluster sample fractions, and per-sample outlier flags to an output table (e.g. sample_outliers.csv) with a sample identifier field and an outlier flag field.

## Related tools

- **msiFlow** (Provides the automated workflow implementing the UMAP + HDBSCAN outlier detection principle for multi-sample MALDI MSI pre-processing) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Automates and parallelizes the outlier-detection workflow steps for high-throughput multi-sample analyses)
- **Docker** (Packages msiFlow workflows (including pre-processing with outlier detection) into a Docker image for platform-independent reuse)

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git && snakemake --use-conda --cores 1 msi_preprocessing_flow
```

## Evaluation signals

- sample_outliers.csv exists and contains a sample identifier field and an outlier flag field.
- The set of flagged outlier samples equals those whose pixel fraction in SSC-dominated clusters exceeds the defined 70% threshold.
- Per-cluster sample fractions in the output sum to consistent pixel counts, and every SSC cluster has >=70% of pixels from one sample.
- The HDBSCAN clustering is computed on the joint (all-sample) UMAP embedding, not per-sample embeddings — pixels from different samples must be comparable in the same 2D space.
- Re-running the workflow on a known-good multi-sample dataset (e.g. Zenodo 10.5281/zenodo.11913042 example data, processable with 8 GB RAM) reproduces the expected outlier flags.

## Limitations

- The threshold of 70% pixels from a single sample defining a sample-specific cluster is a fixed heuristic; borderline samples near this cutoff may be inconsistently flagged.
- Outlier detection is effective only for multi-sample datasets because it relies on spectral similarity across samples; it cannot flag outliers in single-sample acquisitions.
- UMAP is stochastic; unless random seeds are fixed, cluster boundaries and hence outlier flags may vary between runs.

## Evidence
