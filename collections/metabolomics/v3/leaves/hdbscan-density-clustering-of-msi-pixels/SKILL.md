---
name: hdbscan-density-clustering-of-msi-pixels
description: Use when UMAP-embedded MALDI MSI pixels (e.g., from imzML data) must
  be grouped into spectrally coherent clusters without a predefined cluster count
  — e.g., for automated matrix/off-tissue pixel detection, tissue segmentation, or
  flagging sample outliers in multi-sample datasets.
when_to_use_negative:
- Segmenting immunofluorescence microscopy (IFM) marker images — msiFlow handles those
  with thresholding-based if_segmentation_flow, not UMAP/HDBSCAN clustering.
- Outlier flagging with only one sample available — the SSC criterion exploits spectral
  similarity across multiple samples.
- Assigning known molecular annotations to clusters — the article states msiFlow does
  not include an annotation or interpretation step.
edam_operation: http://edamontology.org/operation_3432
tools:
- name: msiFlow
  role: Implements msi_segmentation_flow (dim. reduction and clustering of MSI data)
    plus the border-connected matrix-detection and SSC-based outlier-detection modules
    built on HDBSCAN clusters.
  repo: github.com/Immunodynamics-Engel-Lab/msiflow
- name: UMAP
  role: Projects pixel-wise spectra to a shared 2D embedding (per sample for matrix
    detection; jointly across samples for outlier detection) prior to HDBSCAN.
- name: HDBSCAN
  role: Density-based clustering of the 2D UMAP-embedded pixels; its clusters feed
    border-connected matrix detection and the 70%-threshold SSC outlier flagging.
- name: imzy
  role: Reader interface used to load raw imzML pixel spectra before dimensionality
    reduction.
  repo: github.com/vandeplaslab/imzy
- name: Snakemake
  role: Automates the msi_segmentation_flow steps so UMAP+HDBSCAN clustering runs
    reproducibly and in parallel.
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:539aea29f48f06aca280b69304896c0c898b3c4150ac35529bae2f99a68aa2c6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Immunodynamics-Engel-Lab/msiflow
  tools:
  - Docker
  - HDBSCAN
  - Snakemake
  - UMAP
  - imzy
  - msiFlow
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# hdbscan-density-clustering-of-msi-pixels

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

HDBSCAN density-based clustering of UMAP-embedded MALDI MSI pixels, as automated in msiFlow's msi_segmentation_flow, partitions pixels into spectrally coherent clusters without a predefined cluster count. These clusters are the basis of msiFlow's automated matrix/off-tissue pixel detection and its multi-sample outlier identification via sample-specific clusters (SSC).

## When to use

Use when UMAP-embedded MALDI MSI pixels (e.g., from imzML data) must be grouped into spectrally coherent clusters without a predefined cluster count — e.g., for automated matrix/off-tissue pixel detection, tissue segmentation, or flagging sample outliers in multi-sample datasets.

## When NOT to use

- Segmenting immunofluorescence microscopy (IFM) marker images — msiFlow handles those with thresholding-based if_segmentation_flow, not UMAP/HDBSCAN clustering.
- Outlier flagging with only one sample available — the SSC criterion exploits spectral similarity across multiple samples.
- Assigning known molecular annotations to clusters — the article states msiFlow does not include an annotation or interpretation step.

## Inputs

- Raw MALDI MSI pixel spectra in imzML format (.imzML/.ibd), e.g., MALDI-2 or t-MALDI-2 acquisitions
- Pixel-wise peak features / 2D UMAP embedding of MSI pixels (per-sample for matrix detection; joint across samples for outlier detection)

## Outputs

- HDBSCAN cluster label per pixel per sample in the 2D UMAP space
- Binary matrix/off-tissue mask per sample saved as matrix_mask.tif (post-processed morphologically)
- sample_outliers.csv with cluster assignments, per-cluster sample fractions, sample identifiers, and outlier flags

## How to apply

Load raw imzML pixel spectra (e.g., MALDI-2 / t-MALDI-2 acquisitions) with the imzy reader and embed pixel-wise peak features in 2D with UMAP — embedding each sample separately for matrix detection, or all samples jointly so pixels from different samples are comparable for outlier detection. Cluster the 2D-embedded pixels with HDBSCAN, the density-based step of msiFlow's msi_segmentation_flow ('dim. reduction and clustering of MSI data'); density clustering is used because the number of clusters (tissue regions, matrix, off-tissue) is unknown in advance. For matrix detection, designate the HDBSCAN cluster connected to the largest number of border points of the measured area as the off-tissue/matrix cluster, extend it by merging remaining clusters whose mean spectra correlate with it, and apply morphological post-processing (closing/smoothing) to obtain the binary mask. For multi-sample outlier detection, compute per-cluster sample composition, label clusters in which >=70% of pixels originate from one sample as sample-specific clusters (SSC), and flag samples whose pixels are dominated by SSC pixels as outliers, writing cluster assignments, per-cluster sample fractions and outlier flags to sample_outliers.csv. The rationale: border connectivity works whole-sample because matrix surrounds the tissue (reliance on tissue architecture and borders), while SSC-based flagging is effective on multi-sample datasets because it relies on spectral similarity across samples.

## Related tools

- **msiFlow** (Implements msi_segmentation_flow (dim. reduction and clustering of MSI data) plus the border-connected matrix-detection and SSC-based outlier-detection modules built on HDBSCAN clusters.) — github.com/Immunodynamics-Engel-Lab/msiflow
- **UMAP** (Projects pixel-wise spectra to a shared 2D embedding (per sample for matrix detection; jointly across samples for outlier detection) prior to HDBSCAN.)
- **HDBSCAN** (Density-based clustering of the 2D UMAP-embedded pixels; its clusters feed border-connected matrix detection and the 70%-threshold SSC outlier flagging.)
- **imzy** (Reader interface used to load raw imzML pixel spectra before dimensionality reduction.) — github.com/vandeplaslab/imzy
- **Snakemake** (Automates the msi_segmentation_flow steps so UMAP+HDBSCAN clustering runs reproducibly and in parallel.)

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git && snakemake msi_segmentation_flow
```

## Evaluation signals

- sample_outliers.csv exists and contains a sample identifier field and an outlier flag field; the set of flagged samples equals those whose pixel fraction in SSC-dominated clusters exceeds the 70% threshold.
- For matrix detection, the selected HDBSCAN cluster is the border-connected cluster containing the most pixels, and the post-processed mask forms a coherent matrix shell around the measured tissue area while excluding tissue-covered pixels.
- Every pixel receives exactly one HDBSCAN cluster label (or noise) per run, and labels derive from the 2D UMAP embedding rather than pixel coordinates.
- Clusters labelled sample-specific (SSC) have >=70% of their pixels originating from a single sample when per-cluster sample composition is recomputed.

## Limitations

- Segmentation example data requires at least 120 GB RAM (the paper notes 8 GB RAM suffices only for workflows other than MSI pre-processing and segmentation), so smaller demo data may need to be substituted.
- msiFlow does not include an annotation or interpretation step — clusters surface interesting molecular patterns and preliminary candidates, not annotated molecular identities.
- Automated matrix detection relies on tissue architecture and borders and is framed for whole-sample measurements; outlier detection presumes a multi-sample dataset since it relies on spectral similarity across samples.

## Evidence

- [full_text] msiFlow reduces each sample's MALDI-2 MSI data to two dimensions with UMAP followed by HDBSCAN clustering: [evidence span withheld — anchor status: not_found]
- [full_text] the cluster connected to most border points of the measured area is designated as the off-tissue/matrix cluster: [evidence span withheld — anchor status: not_found]
- [full_text] defining sample-specific clusters (SSC) as clusters in which 70% of pixels originate from one sample: [evidence span withheld — anchor status: not_found]
- [readme] Read, inspect, process, and export imaging mass spectrometry datasets from Python.: [evidence span withheld — anchor status: external_doc]
