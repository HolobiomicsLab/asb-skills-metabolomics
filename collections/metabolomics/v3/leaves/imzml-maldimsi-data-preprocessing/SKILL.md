---
name: imzml-maldimsi-data-preprocessing
description: Use when raw, multi-vendor MALDI MSI data (e.g. imzML from MALDI-2/t-MALDI-2
  acquisitions) must be turned into a pixel-wise peak list and non-informative or
  exogenous ions removed by spatial-coherence filtering before intensity normalisation.
when_to_use_negative:
- Input is already a peak-picked or feature table and the goal is downstream segmentation,
  registration, or ROI statistics — use msi_segmentation_flow, msi_if_registration_flow,
  or region_group_analysis_flow instead.
- The goal is lipid annotation or biological interpretation of candidates — msiFlow
  does not include an annotation or interpretation step, so pre-processing cannot
  deliver it.
- The data are immunofluorescence microscopy (IFM) images rather than MALDI MSI spectra
  — IFM has its own thresholding-based if_segmentation_flow.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: msiFlow
  role: Hosts the Snakemake msi_preprocessing_flow that performs pixel-wise peak picking
    and the spatial-coherence peak filtering step on raw MALDI MSI data
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine that automates all msiFlow pre-processing steps and enables
    parallel high-throughput processing
- name: Docker
  role: Packages the msiFlow workflows into a Docker image for platform-independent,
    non-programmer-friendly execution
- name: imzy
  role: Reader/writer interface for imzML and other imaging mass spectrometry formats
    cited in the article's methods for imaging-MS data access
  repo: https://github.com/vandeplaslab/imzy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:fc3ada5b113b728d51a056e727ce17c489031c0138f82dbdf0127a6b24cfb068
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# imzml-maldimsi-data-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs msiFlow's Snakemake-based msi_preprocessing_flow to turn raw, multi-vendor MALDI MSI data (e.g. MALDI-2 lipidomic acquisitions) into a pixel-wise peak-picked peak list, then removes non-informative ions via a spatial-coherence peak filtering step applied before intensity normalisation.

## When to use

Use when raw, multi-vendor MALDI MSI data (e.g. imzML from MALDI-2/t-MALDI-2 acquisitions) must be turned into a pixel-wise peak list and non-informative or exogenous ions removed by spatial-coherence filtering before intensity normalisation.

## When NOT to use

- Input is already a peak-picked or feature table and the goal is downstream segmentation, registration, or ROI statistics — use msi_segmentation_flow, msi_if_registration_flow, or region_group_analysis_flow instead.
- The goal is lipid annotation or biological interpretation of candidates — msiFlow does not include an annotation or interpretation step, so pre-processing cannot deliver it.
- The data are immunofluorescence microscopy (IFM) images rather than MALDI MSI spectra — IFM has its own thresholding-based if_segmentation_flow.

## Inputs

- Raw multi-vendor MALDI MSI acquisition data (e.g. imzML/vendor formats; MALDI-2/t-MALDI-2 example data from Zenodo DOI 10.5281/zenodo.11913042)
- Pixel-wise peak-picked spectra/peak list derived from the raw data (input to the spatial-coherence filtering step)
- Workflow configuration (config.yaml) specifying the spatial coherence threshold

## Outputs

- Peak list generated from raw MALDI MSI data by msi_preprocessing_flow
- filtered_peak_list.csv: filtered peak table with ion m/z and intensity columns containing only ions that pass the spatial coherence threshold

## How to apply

Clone msiFlow (git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git) and run its Snakemake pre-processing workflow (msi_preprocessing_flow), or use the packaged Docker image, on the raw multi-vendor MALDI MSI acquisition — e.g. the example data on Zenodo (10.5281/zenodo.11913042) — which performs pixel-wise peak picking to generate a peak list from the raw spectra. Because standard peak picking alone retains non-informative or exogenous signals, msiFlow adds a peak filtering step: compute the spatial coherence metric for each ion from its intensity distribution across pixels, then remove all ions whose score falls below the coherence threshold configured in the workflow's config.yaml. Apply this filtering immediately before intensity normalisation so only spatially coherent, informative ions are scaled downstream. Assemble the surviving ions into a filtered peak table (filtered_peak_list.csv) with ion m/z and intensity columns.

## Related tools

- **msiFlow** (Hosts the Snakemake msi_preprocessing_flow that performs pixel-wise peak picking and the spatial-coherence peak filtering step on raw MALDI MSI data) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine that automates all msiFlow pre-processing steps and enables parallel high-throughput processing)
- **Docker** (Packages the msiFlow workflows into a Docker image for platform-independent, non-programmer-friendly execution)
- **imzy** (Reader/writer interface for imzML and other imaging mass spectrometry formats cited in the article's methods for imaging-MS data access) — https://github.com/vandeplaslab/imzy

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git
```

## Evaluation signals

- filtered_peak_list.csv exists and parses as a readable tabular file with ion m/z and intensity columns.
- Every retained ion's spatial coherence score meets or exceeds the threshold configured in config.yaml; recomputing the metric on the output confirms this.
- The filtered peak list is a strict subset of the pixel-wise peak-picked input list — ions are only removed, none added.
- Running msi_preprocessing_flow on the Zenodo example data (10.5281/zenodo.11913042) reproduces the expected output files shipped with the msiFlow demo dataset.
- Pipeline ordering holds: peak filtering runs after pixel-wise peak picking and immediately before intensity normalisation.

## Limitations

- The spatial coherence threshold is configured per dataset in config.yaml and no universal cutoff is given, so threshold choice directly controls which ions survive.
- msiFlow stops at identifying interesting molecular patterns and preliminary candidates: it does not include an annotation or interpretation step.
- Scaling beyond MALDI MSI lipidomics to metabolomics and proteomics modalities is anticipated but not demonstrated in the paper.

## Evidence
