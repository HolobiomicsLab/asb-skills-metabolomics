---
name: maldi-msi-spectral-preprocessing
description: Use when raw vendor MALDI MSI files must be converted into a reproducible,
  quality-controlled peak list (e.g. imzML output with QC visualisations) before segmentation,
  registration or statistical analysis.
when_to_use_negative:
- Data is already a processed peak list or feature table — use downstream msiFlow
  workflows (segmentation, analysis) instead
- The goal is lipid annotation/interpretation — msiFlow explicitly does not include
  an annotation or interpretation step
- Imaging data is IFM (immunofluorescence microscopy) rather than MALDI MSI — use
  if_segmentation_flow or msi_if_registration_flow instead
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0121
tools:
- name: msiFlow
  role: Provides the msi_preprocessing_flow Snakemake workflow that performs peak
    picking, spatial-coherence peak filtering, alignment, matrix pixel removal, normalisation,
    outlier detection and de-isotoping
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine executing msi_preprocessing_flow with parallel, high-throughput
    data processing and optional memory resource limits
- name: Docker
  role: Containerised distribution of msiFlow (image phispa1812/msiflow_cli) so the
    pre-processing workflow can be run without local installation
- name: imzy
  role: Python reader/writer interface for imzML and other imaging mass spectrometry
    formats cited by the paper
  repo: https://github.com/vandeplaslab/imzy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:dc1c1d58ee1abd2042cbc9389ce20c96e4d7e2b4d6bde0074bf88c4f1120afc4
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
  - imzy
  - msiFlow
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# maldi-msi-spectral-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated pre-processing of raw, multi-vendor MALDI MSI data into a processed peak list using msiFlow's Snakemake-based msi_preprocessing_flow, covering pixel-wise peak picking with spatial-coherence filtering, alignment, matrix pixel removal, normalisation, outlier detection and de-isotoping.

## When to use

Use when raw vendor MALDI MSI files must be converted into a reproducible, quality-controlled peak list (e.g. imzML output with QC visualisations) before segmentation, registration or statistical analysis.

## When NOT to use

- Data is already a processed peak list or feature table — use downstream msiFlow workflows (segmentation, analysis) instead
- The goal is lipid annotation/interpretation — msiFlow explicitly does not include an annotation or interpretation step
- Imaging data is IFM (immunofluorescence microscopy) rather than MALDI MSI — use if_segmentation_flow or msi_if_registration_flow instead

## Inputs

- Raw vendor MALDI MSI files (multi-vendor raw imaging data, e.g. Bruker/Waters formats readable via imzy)
- config.yaml with pre-processing parameters
- Example dataset from Zenodo deposit 10.5281/zenodo.11913042 (requires ≥120 GB RAM)

## Outputs

- Processed peak list generated from the raw MALDI MSI data
- Processed data in the open imzML standard format
- Quality-control visualisations

## How to apply

Download the example dataset from Zenodo (10.5281/zenodo.11913042; the pre-processing example requires at least 120 GB RAM) and obtain msiFlow either via Docker (docker pull phispa1812/msiflow_cli) or by cloning github.com/Immunodynamics-Engel-Lab/msiflow and creating the conda environment from msiflow_env.yaml. Follow the README in the msi_preprocessing_flow directory to structure and name the raw MALDI MSI input files and set parameters in config.yaml. Run the workflow with Snakemake, pointing at msi_preprocessing_flow/Snakefile and the config file, optionally constraining memory with --resources mem_mb=<available_RAM_in_MB> to prevent out-of-memory errors; Snakemake processes raw vendor files in parallel for high-throughput runs. The pipeline performs pixel-wise peak picking (including an additional peak filtering step that computes the spatial coherence metric on spatial information), peak alignment, automated matrix pixel removal (UMAP/HDBSCAN relying on tissue architecture and borders), normalisation, outlier detection based on spectral similarity across multiple samples, and de-isotoping to generate a peak list from the raw data. Outputs are written in the open imzML standard together with quality-control visualisations; verify the peak list file exists at the locations described in the workflow README.

## Related tools

- **msiFlow** (Provides the msi_preprocessing_flow Snakemake workflow that performs peak picking, spatial-coherence peak filtering, alignment, matrix pixel removal, normalisation, outlier detection and de-isotoping) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine executing msi_preprocessing_flow with parallel, high-throughput data processing and optional memory resource limits)
- **Docker** (Containerised distribution of msiFlow (image phispa1812/msiflow_cli) so the pre-processing workflow can be run without local installation)
- **imzy** (Python reader/writer interface for imzML and other imaging mass spectrometry formats cited by the paper) — https://github.com/vandeplaslab/imzy

## Examples

```
snakemake --snakefile msi_preprocessing_flow/Snakefile --cores <number-of-cores> --configfile <path-to-config> --resources mem_mb=<available_RAM_in_MB>
```

## Evaluation signals

- The Snakemake workflow completes without error and a peak list output file is generated from the raw MALDI MSI data (the paper's stated qualitative check)
- Generated output files appear at the locations described in the msi_preprocessing_flow README
- Processed data is emitted in imzML format (plus companion binary) that a reader such as imzy can open
- QC visualisations are produced alongside the peak list, allowing inspection of peak picking, alignment and matrix-removal quality
- Runs on the ≥120 GB RAM example dataset complete without out-of-memory failure when --resources mem_mb is set to available RAM

## Limitations

- The pre-processing example dataset requires at least 120 GB RAM; without the --resources mem_mb constraint, out-of-memory errors may occur
- No quantitative acceptance threshold is reported for this step; validation is qualitative (workflow completes and a peak list is generated)
- So far msiFlow does not include an annotation or interpretation step
- The paper anticipates scaling to other MSI modalities (metabolomics, proteomics) but has not demonstrated it

## Evidence
