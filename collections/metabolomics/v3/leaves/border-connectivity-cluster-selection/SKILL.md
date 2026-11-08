---
name: border-connectivity-cluster-selection
description: Use when pre-processing whole-sample MALDI MSI measurements where matrix/off-tissue
  pixels must be identified automatically without defining known matrix peaks in advance.
when_to_use_negative:
- You already have a curated list of known matrix peaks to subtract — direct peak
  filtering suffices and border-connectivity detection is unnecessary
- The dataset is not a whole-sample measurement — the article notes the approach's
  applicability relies on tissue architecture and borders, so cropped/tissue-only
  regions lack the matrix shell it detects
- Performing the msi_segmentation_flow tissue segmentation itself — this skill targets
  matrix detection during pre-processing, not downstream dim. reduction/clustering
  for tissue segmentation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3620
- http://edamontology.org/topic_0121
tools:
- name: msiFlow
  role: Snakemake workflow framework implementing the automated matrix detection during
    MALDI MSI pre-processing
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: imzy
  role: Python reader interface used to load raw imzML pixel spectra before UMAP embedding
  repo: https://github.com/vandeplaslab/imzy
- name: Snakemake
  role: Workflow engine automating and parallelising the matrix detection steps
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:280fc7795a00c2448ff8328a8915fb7665543f7e4e97bd37011dbed25a72fce5
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
  - Snakemake
  - imzy
  - msiFlow
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3620
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# border-connectivity-cluster-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An unsupervised matrix/off-tissue pixel detection method for MALDI MSI in which UMAP-embedded pixel spectra are clustered with HDBSCAN and the cluster connected to the most border pixels of the measured area is designated as the matrix cluster.

## When to use

Use when pre-processing whole-sample MALDI MSI measurements where matrix/off-tissue pixels must be identified automatically without defining known matrix peaks in advance.

## When NOT to use

- You already have a curated list of known matrix peaks to subtract — direct peak filtering suffices and border-connectivity detection is unnecessary
- The dataset is not a whole-sample measurement — the article notes the approach's applicability relies on tissue architecture and borders, so cropped/tissue-only regions lack the matrix shell it detects
- Performing the msi_segmentation_flow tissue segmentation itself — this skill targets matrix detection during pre-processing, not downstream dim. reduction/clustering for tissue segmentation

## Inputs

- Raw MALDI MSI pixel spectra in imzML format (.imzML/.ibd)
- Measured-area geometry / pixel grid for determining border pixels

## Outputs

- Post-processed binary matrix mask per sample (matrix_mask.tif)
- HDBSCAN cluster labels over UMAP-embedded pixels

## How to apply

Read the raw imzML pixel spectra with the imzy reader, then reduce each sample's pixel spectra to a 2-dimensional UMAP embedding that groups pixels by spectral similarity. Cluster the embedded pixels with HDBSCAN density-based clustering. Select as the matrix cluster the HDBSCAN cluster whose region is connected to the largest number of border pixels of the measured area — the rationale is that matrix surrounds the tissue, so the matrix cluster must touch the measurement borders. Extend the selected cluster by merging remaining clusters whose mean spectra correlate with it, then apply morphological post-processing (closing/smoothing) to the resulting binary mask. Save the post-processed mask per sample as matrix_mask.tif and visually verify it forms a coherent matrix shell around the tissue.

## Related tools

- **msiFlow** (Snakemake workflow framework implementing the automated matrix detection during MALDI MSI pre-processing) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **imzy** (Python reader interface used to load raw imzML pixel spectra before UMAP embedding) — https://github.com/vandeplaslab/imzy
- **Snakemake** (Workflow engine automating and parallelising the matrix detection steps) — https://github.com/Immunodynamics-Engel-Lab/msiflow

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git && snakemake --use-singularity --cores 1 -s workflows/msi_preprocessing_flow/Snakefile
```

## Evaluation signals

- The selected cluster is the HDBSCAN cluster connected to the largest number of border pixels of the measured area and among these contains the most pixels
- matrix_mask.tif exists per sample and the mask forms a coherent matrix shell surrounding the measured tissue area
- Visually, the mask covers matrix regions around the tissue and excludes tissue-covered pixels
- Cluster extension by mean-spectrum correlation merges only clusters spectrally similar to the selected matrix cluster, and morphological closing/smoothing yields a smooth binary mask

## Limitations

- Segmentation example data requires at least 120 GB RAM, so large whole-sample runs need substantial memory
- The approach is justified for whole-sample measurements because it relies on tissue architecture and borders; it is not demonstrated for tissue-cropped regions
- The method is not a matrix-peak-list generator — it produces a pixel mask, not annotated matrix m/z signals

## Evidence

- [discussion] Automated matrix pixel detection via UMAP and HDBSCAN using tissue architecture and borders: "The automated matrix detection approach is applicable for whole-sample measurements due to its reliance on tissue architecture and borders"
- [methods] border-connected cluster selection: [evidence span withheld — anchor status: not_found]
- [discussion] workflow automation context: "All steps are automated in Snakemake workflows, which enables parallel data processing for high-throughput analyses."
- [readme] imzy reader for raw imzML spectra: [evidence span withheld — anchor status: external_doc]
- [readme] segmentation workflow description: "| segmentation | dim. reduction and clustering of MSI data | `msi_segmentation_flow`|"
