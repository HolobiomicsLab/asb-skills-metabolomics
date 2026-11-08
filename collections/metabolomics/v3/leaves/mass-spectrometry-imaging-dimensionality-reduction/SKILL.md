---
name: mass-spectrometry-imaging-dimensionality-reduction
description: Use when pixel-wise peak features from one or multiple MALDI MSI samples
  must be jointly projected into a low-dimensional embedding (UMAP) and density-clustered
  (HDBSCAN) for segmentation or sample-outlier flagging.
when_to_use_negative:
- Data are already segmented or already a feature/cluster table, so no pixel-level
  dimensionality reduction is needed
- Single-sample datasets where outlier detection relying on spectral similarity across
  multiple samples cannot be applied
- Annotation or interpretation of candidate molecules, which msiFlow explicitly does
  not include
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: msiFlow
  role: Provides the automated Snakemake workflows (msi_segmentation_flow and related
    UMAP-based workflows) that perform joint UMAP dimensionality reduction, HDBSCAN
    clustering, sample-specific cluster (SSC) definition, and multi-sample outlier
    identification
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine automating and parallelizing the dimensionality reduction
    and clustering steps for high-throughput analyses
- name: Docker
  role: Containerizes all msiFlow workflows, including the UMAP/HDBSCAN segmentation
    flow, for platform-independent reproducible execution
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:d19767d44a579407eb4b99595136be1b1ba995cee34662d39d9925c29fdb4a6b
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# mass-spectrometry-imaging-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Joint UMAP-based dimensionality reduction of pixel-wise peak features from MALDI MSI data, coupled with HDBSCAN density clustering, as implemented in msiFlow's msi_segmentation_flow for segmentation, matrix pixel detection, and multi-sample outlier identification.

## When to use

Use when pixel-wise peak features from one or multiple MALDI MSI samples must be jointly projected into a low-dimensional embedding (UMAP) and density-clustered (HDBSCAN) for segmentation or sample-outlier flagging.

## When NOT to use

- Data are already segmented or already a feature/cluster table, so no pixel-level dimensionality reduction is needed
- Single-sample datasets where outlier detection relying on spectral similarity across multiple samples cannot be applied
- Annotation or interpretation of candidate molecules, which msiFlow explicitly does not include

## Inputs

- Raw MALDI MSI data (e.g. imzML) from one or multiple samples
- Pixel-wise peak feature tables extracted during msi_preprocessing_flow
- Multi-sample example data from Zenodo (DOI 10.5281/zenodo.11913042) or the msiFlow demo dataset

## Outputs

- 2D UMAP embedding of all pixels in a shared low-dimensional space
- HDBSCAN cluster assignments per pixel
- Per-cluster sample composition fractions and sample-specific cluster (SSC) labels
- sample_outliers.csv with a sample identifier field and an outlier flag field

## How to apply

Extract pixel-wise peak features from all samples and jointly reduce them to two dimensions with UMAP so pixels from different samples become comparable in a shared embedding. Cluster the joint UMAP embedding with HDBSCAN, the same density-based clustering strategy msiFlow uses for its UMAP-based workflows such as msi_segmentation_flow (dim. reduction and clustering of MSI data) and automated matrix pixel detection. For multi-sample outlier detection, compute per-cluster sample composition and label any cluster in which >=70% of pixels originate from one sample as a sample-specific cluster (SSC); samples whose pixels are dominated by SSC clusters are flagged as outliers, exploiting spectral similarity across multiple samples. Run the workflow via msiFlow's Snakemake pipelines inside the provided Docker image for reproducibility, using the Zenodo example data (DOI 10.5281/zenodo.11913042).

## Related tools

- **msiFlow** (Provides the automated Snakemake workflows (msi_segmentation_flow and related UMAP-based workflows) that perform joint UMAP dimensionality reduction, HDBSCAN clustering, sample-specific cluster (SSC) definition, and multi-sample outlier identification) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine automating and parallelizing the dimensionality reduction and clustering steps for high-throughput analyses)
- **Docker** (Containerizes all msiFlow workflows, including the UMAP/HDBSCAN segmentation flow, for platform-independent reproducible execution)

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git
```

## Evaluation signals

- sample_outliers.csv exists and contains a sample identifier field and an outlier flag field
- The set of flagged outlier samples equals those whose pixel fraction in sample-specific clusters exceeds the 70% threshold
- Every pixel in the joint UMAP embedding receives an HDBSCAN cluster assignment
- Cluster labels and per-cluster sample fractions are internally consistent (sample fractions per cluster sum to 1)
- The workflow runs end-to-end in the msiFlow Snakemake/Docker environment on the Zenodo example data

## Limitations

- Outlier detection requires multi-sample datasets, as it relies on spectral similarity across multiple samples
- Example data for workflows other than MSI pre-processing and segmentation is stated to be processable on a computer with 8 GB RAM; full pre-processing/segmentation may require more resources

## Evidence

- [full_text] msiFlow's outlier identification reduces the data of all samples jointly to two dimensions via UMAP and clusters them with HDBSCAN: [evidence span withheld — anchor status: not_found]
