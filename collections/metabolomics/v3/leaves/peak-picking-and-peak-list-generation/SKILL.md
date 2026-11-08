---
name: peak-picking-and-peak-list-generation
description: Use when raw vendor MALDI MSI data must be converted into a centroided
  peak list with peak filtering, alignment, matrix pixel removal, normalisation, outlier
  detection and de-isotoping before segmentation or ROI-level analysis.
when_to_use_negative:
- Data are already a processed peak list or feature table — use msi_segmentation_flow
  (dimensionality reduction and clustering) or region_group_analysis_flow instead
  of re-running pre-processing
- The goal is registering immunofluorescence microscopy (IFM) to MSI or thresholding-based
  segmentation of IFM markers — use msi_if_registration_flow or if_segmentation_flow
  rather than msi_preprocessing_flow
- Lipid or molecular annotation/interpretation of peaks is the required output — msiFlow
  does not include an annotation or interpretation step
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: msiFlow
  role: Hosts the msi_preprocessing_flow Snakemake workflow that performs pixel-wise
    peak picking, spatial-coherence peak filtering, alignment, matrix pixel removal
    (UMAP/HDBSCAN), normalisation, outlier detection and de-isotoping to generate
    the peak list
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine that automates and parallelises the pre-processing steps for
    high-throughput analyses and enforces memory limits via --resources mem_mb
- name: Docker
  role: Container delivery of msiFlow (image phispa1812/msiflow_cli) so the pre-processing
    workflow runs without local environment setup, making it suitable for non-programmers
- name: imzy
  role: Reader/writer interface for imzML and other imaging mass spectrometry formats
    cited in msiFlow's methods, supporting imzML input/output of the pre-processed
    data
  repo: https://github.com/vandeplaslab/imzy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:c2d488a616118cf8ec6af7c6f2e3c1aa64ef861270761c1b43edf234bba26617
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# peak-picking-and-peak-list-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

The msi_preprocessing_flow of msiFlow converts raw vendor MALDI MSI files into a peak list via pixel-wise peak picking with a spatial-coherence-based peak filtering step, followed by alignment, matrix pixel removal, normalisation, spectral-similarity outlier detection and de-isotoping. It exists because existing MALDI MSI software is incomplete and labour-intensive, and packaging this in Snakemake makes it reproducible and parallelisable.

## When to use

Use when raw vendor MALDI MSI data must be converted into a centroided peak list with peak filtering, alignment, matrix pixel removal, normalisation, outlier detection and de-isotoping before segmentation or ROI-level analysis.

## When NOT to use

- Data are already a processed peak list or feature table — use msi_segmentation_flow (dimensionality reduction and clustering) or region_group_analysis_flow instead of re-running pre-processing
- The goal is registering immunofluorescence microscopy (IFM) to MSI or thresholding-based segmentation of IFM markers — use msi_if_registration_flow or if_segmentation_flow rather than msi_preprocessing_flow
- Lipid or molecular annotation/interpretation of peaks is the required output — msiFlow does not include an annotation or interpretation step

## Inputs

- Raw vendor MALDI MSI data files, structured and named per the msi_preprocessing_flow README (vendor-neutral import in msiFlow)
- config.yaml configuration file with pre-processing parameters
- Example dataset from Zenodo 10.5281/zenodo.11913042 (requires at least 120 GB RAM) or the msiFlow demo dataset at github.com/Immunodynamics-Engel-Lab/msiflow/tree/main/demo

## Outputs

- Peak list file generated from the raw MALDI MSI data
- Processed data in the open standard imzML format
- Quality control visualisations of the pre-processing steps

## How to apply

Obtain raw MALDI MSI inputs (e.g. the Zenodo example dataset 10.5281/zenodo.11913042, which needs at least 120 GB RAM, or the smaller msiFlow demo dataset) and set up msiFlow either via Docker (docker pull phispa1812/msiflow_cli) or locally by cloning github.com/Immunodynamics-Engel-Lab/msiflow and creating the conda environment from msiflow_env.yaml. Structure and name the raw input files and set the parameters in config.yaml exactly as documented in the README of the msi_preprocessing_flow directory, because the workflow expects a specific input layout. Launch the pipeline with Snakemake (`snakemake --snakefile msi_preprocessing_flow/Snakefile --cores <number-of-cores> --configfile <path-to-config>`), optionally adding `--resources mem_mb=<available_RAM_in_MB>` to prevent out-of-memory errors on large datasets. The workflow performs pixel-wise peak picking and utilises the spatial information for peak filtering by computing the spatial coherence metric, removing non-informative or exogenous signals (such as matrix) beyond what standard peak picking achieves. Subsequent steps perform alignment, automated matrix pixel detection via UMAP and HDBSCAN (relying on tissue architecture and borders), normalisation, outlier detection based on spectral similarity across multiple samples, and de-isotoping, with all steps automated and parallelised by Snakemake. Validate qualitatively — the paper reports no quantitative acceptance threshold — by confirming the workflow completes, a peak list is generated, and output files appear at the locations described in the workflow README alongside quality control visualisations.

## Related tools

- **msiFlow** (Hosts the msi_preprocessing_flow Snakemake workflow that performs pixel-wise peak picking, spatial-coherence peak filtering, alignment, matrix pixel removal (UMAP/HDBSCAN), normalisation, outlier detection and de-isotoping to generate the peak list) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine that automates and parallelises the pre-processing steps for high-throughput analyses and enforces memory limits via --resources mem_mb)
- **Docker** (Container delivery of msiFlow (image phispa1812/msiflow_cli) so the pre-processing workflow runs without local environment setup, making it suitable for non-programmers)
- **imzy** (Reader/writer interface for imzML and other imaging mass spectrometry formats cited in msiFlow's methods, supporting imzML input/output of the pre-processed data) — https://github.com/vandeplaslab/imzy

## Examples

```
snakemake --snakefile msi_preprocessing_flow/Snakefile --cores <number-of-cores> --configfile <path-to-config> --resources mem_mb=<available_RAM_in_MB>
```

## Evaluation signals

- The Snakemake run completes without errors and a peak list file is generated from the raw MALDI MSI data — the paper's qualitative acceptance criterion ('generates a peak list from raw MALDI MSI data')
- Generated output files are found at the locations described in the msi_preprocessing_flow workflow README
- Quality control visualisations are produced alongside the processed imzML output
- Matrix pixels are removed: automated matrix detection via UMAP and HDBSCAN should exclude non-tissue regions by relying on tissue architecture and borders
- No out-of-memory failures when --resources mem_mb is set to available RAM (the Zenodo example dataset requires at least 120 GB RAM)

## Limitations

- The example dataset on Zenodo (10.5281/zenodo.11913042) requires at least 120 GB RAM; omitting --resources mem_mb can cause out-of-memory errors during pre-processing
- The paper reports no quantitative acceptance threshold for this pre-processing step; validation is qualitative (workflow completes and a peak list is generated)
- msiFlow does not include an annotation or interpretation step, so the generated peak list still requires downstream annotation (e.g. mass matching to databases such as LipidMaps) elsewhere

## Evidence
