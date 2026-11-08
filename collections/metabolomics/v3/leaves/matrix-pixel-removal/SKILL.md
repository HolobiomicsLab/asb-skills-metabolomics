---
name: matrix-pixel-removal
description: Use when pre-processing raw multi-vendor MALDI MSI data (vendor formats
  converted to imzML) where matrix/spray pixels would contaminate peak lists, and
  the sample must be segmented into tissue vs. matrix regions automatically without
  manual ROI drawing.
when_to_use_negative:
- The task is segmentation or IFM registration rather than MALDI MSI pre-processing
  — use msi_segmentation_flow, if_segmentation_flow or msi_if_registration_flow instead
- The input is already a peak list or processed imzML peak matrix, not raw vendor
  MALDI MSI data
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: msiFlow
  role: Provides msi_preprocessing_flow, whose automated matrix pixel detection step
    (UMAP + HDBSCAN on tissue architecture and borders) is executed as part of pre-processing
    to produce the peak list
  repo: github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Orchestrates the msi_preprocessing_flow steps, enabling parallel processing
    of raw vendor MSI files and RAM-limited resource scheduling
  repo: github.com/Immunodynamics-Engel-Lab/msiflow
- name: Docker
  role: Alternative deployment of msiFlow CLI (docker pull phispa1812/msiflow_cli)
    for running the pre-processing workflow without local environment setup
  repo: github.com/Immunodynamics-Engel-Lab/msiflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:e501040d1a420f91609097874bc8d42966dbaad4d06f0b13c0349d91c7a78ef5
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
  - Snakemake
  - imzy
  - msiFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# matrix-pixel-removal

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An automated step of the msiFlow msi_preprocessing_flow that detects and removes matrix (MALDI matrix spray) pixels from raw MALDI MSI data using UMAP and HDBSCAN based on tissue architecture and borders, so that downstream peak picking, alignment, normalisation and de-isotoping operate only on true tissue pixels.

## When to use

Use when pre-processing raw multi-vendor MALDI MSI data (vendor formats converted to imzML) where matrix/spray pixels would contaminate peak lists, and the sample must be segmented into tissue vs. matrix regions automatically without manual ROI drawing.

## When NOT to use

- The task is segmentation or IFM registration rather than MALDI MSI pre-processing — use msi_segmentation_flow, if_segmentation_flow or msi_if_registration_flow instead
- The input is already a peak list or processed imzML peak matrix, not raw vendor MALDI MSI data

## Inputs

- Raw multi-vendor MALDI MSI data files (e.g. t-MALDI-2 high-resolution imaging measurements) named/structured per the msi_preprocessing_flow README
- config.yaml with msi_preprocessing_flow parameters
- Example dataset from Zenodo deposit 10.5281/zenodo.11913042 (requires >= 120 GB RAM)

## Outputs

- Peak list generated from raw MALDI MSI data with matrix pixels removed
- Processed data in the open imzML format
- Quality control visualisations from the pre-processing workflow

## How to apply

Run the msiFlow msi_preprocessing_flow Snakemake workflow, which ingests raw MALDI MSI files and executes peak picking, peak filtering via the spatial coherence metric, alignment, matrix pixel removal, normalisation, spectral-similarity-based outlier detection and de-isotoping. Matrix pixels are detected via a UMAP dimensionality reduction followed by HDBSCAN clustering that exploits tissue architecture and tissue borders, making the detection applicable to whole-sample measurements without manually annotated regions. Structure and name the raw MALDI MSI input files per the msi_preprocessing_flow README, set the parameters in config.yaml, and execute with Snakemake (optionally adding --resources mem_mb to avoid out-of-memory errors, since the example dataset requires at least 120 GB RAM). Validate by confirming the workflow completes, the peak list output file is generated, and generated outputs match the locations described in the workflow README.

## Related tools

- **msiFlow** (Provides msi_preprocessing_flow, whose automated matrix pixel detection step (UMAP + HDBSCAN on tissue architecture and borders) is executed as part of pre-processing to produce the peak list) — github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Orchestrates the msi_preprocessing_flow steps, enabling parallel processing of raw vendor MSI files and RAM-limited resource scheduling) — github.com/Immunodynamics-Engel-Lab/msiflow
- **Docker** (Alternative deployment of msiFlow CLI (docker pull phispa1812/msiflow_cli) for running the pre-processing workflow without local environment setup) — github.com/Immunodynamics-Engel-Lab/msiflow

## Examples

```
snakemake --snakefile msi_preprocessing_flow/Snakefile --cores 8 --configfile config.yaml --resources mem_mb=120000
```

## Evaluation signals

- The Snakemake workflow completes without errors for all raw MALDI MSI inputs
- A peak list output file is generated from the raw MALDI MSI data at the location described in the msi_preprocessing_flow README
- Generated output files (peak list, imzML outputs, QC visualisations) match the locations described in the workflow README
- Matrix pixel detection is consistent with tissue architecture and borders across the whole sample (qualitative check, since the paper reports no quantitative acceptance threshold)

## Limitations

- The paper reports no quantitative acceptance threshold for matrix pixel removal or the pre-processing step; validation is qualitative (workflow completes and a peak list is generated)
- Matrix detection relies on tissue architecture and borders, so samples lacking clear tissue borders may be harder to segment
- Running the pre-processing workflow on the deposited example dataset requires at least 120 GB RAM

## Evidence
