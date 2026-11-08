---
name: spatial-coherence-peak-filtering
description: Use when you have pixel-wise peak-picked MALDI MSI spectra (e.g. raw
  vendor data pre-processed by msi_preprocessing_flow) and need to remove non-informative
  or exogenous ion signals before intensity normalisation.
when_to_use_negative:
- Input is raw vendor MALDI MSI data that has not yet undergone pixel-wise peak picking
  — run msi_preprocessing_flow peak picking first
- The goal is lipid annotation or interpretation — msiFlow explicitly does not include
  an annotation or interpretation step
- Data are non-imaging spectra with no spatial (pixel) structure, since the filter
  depends on the spatial distribution of ion intensities across pixels
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: msiFlow
  role: Provides the msi_preprocessing_flow Snakemake workflow that implements pixel-wise
    peak picking and the spatial-coherence-based peak filtering step
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine used to execute the msi_preprocessing_flow containing the
    peak filtering step
- name: imzy
  role: Reader interface for imzML and other imaging mass spectrometry formats used
    to access MALDI MSI data in the workflow
  repo: https://github.com/vandeplaslab/imzy
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:192571c1f6c0ad835d3fa68b0542e2a3f9efb3d48ad2e90900828a2c4c2c1d0a
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spatial-coherence-peak-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A peak-filtering step from msiFlow's msi_preprocessing_flow that removes non-informative or exogenous ions from pixel-wise peak-picked MALDI MSI data by computing a spatial coherence metric from each ion's intensity distribution across pixels and discarding ions below a configured coherence threshold.

## When to use

Use when you have pixel-wise peak-picked MALDI MSI spectra (e.g. raw vendor data pre-processed by msi_preprocessing_flow) and need to remove non-informative or exogenous ion signals before intensity normalisation.

## When NOT to use

- Input is raw vendor MALDI MSI data that has not yet undergone pixel-wise peak picking — run msi_preprocessing_flow peak picking first
- The goal is lipid annotation or interpretation — msiFlow explicitly does not include an annotation or interpretation step
- Data are non-imaging spectra with no spatial (pixel) structure, since the filter depends on the spatial distribution of ion intensities across pixels

## Inputs

- Pixel-wise peak-picked MALDI MSI spectra (peak list with ion m/z and per-pixel intensities)
- msiFlow workflow configuration file (config.yaml) containing the spatial coherence threshold

## Outputs

- Filtered peak list (filtered_peak_list.csv) with ion m/z and intensity columns, retaining only ions passing the spatial coherence threshold

## How to apply

Load the pixel-wise peak-picked spectra generated from raw MALDI MSI data (e.g. the Zenodo deposit 10.5281/zenodo.11913042 example dataset) within the msiFlow pre-processing environment. For each ion in the peak list, compute the spatial coherence metric from its intensity distribution across pixels, leveraging the spatial information of the ion image as msiFlow does on top of standard peak picking. Remove all ions whose spatial coherence score falls below the coherence threshold set in the workflow's config.yaml, thereby eliminating non-informative or exogenous signals. Assemble the surviving ions into a filtered peak table (filtered_peak_list.csv); this step is performed immediately before intensity normalisation in the msi_preprocessing_flow pipeline, which subsequently runs alignment, matrix pixel removal, normalisation, outlier detection and de-isotoping.

## Related tools

- **msiFlow** (Provides the msi_preprocessing_flow Snakemake workflow that implements pixel-wise peak picking and the spatial-coherence-based peak filtering step) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine used to execute the msi_preprocessing_flow containing the peak filtering step)
- **imzy** (Reader interface for imzML and other imaging mass spectrometry formats used to access MALDI MSI data in the workflow) — https://github.com/vandeplaslab/imzy

## Examples

```
snakemake --snakefile msi_preprocessing_flow/Snakefile --cores 8 --configfile config.yaml
```

## Evaluation signals

- filtered_peak_list.csv exists and is a readable tabular file with ion m/z and intensity columns
- Every retained ion in the output passes the spatial coherence threshold configured in config.yaml
- Ions removed correspond to non-informative or exogenous signals (low spatial coherence), checked qualitatively as the paper reports no quantitative acceptance threshold
- The filtering occurs in the pipeline order immediately before intensity normalisation, and the overall msi_preprocessing_flow completes generating a peak list from the raw MALDI MSI data

## Limitations

- The paper reports no quantitative acceptance threshold for this step; validation of the filtering is qualitative (workflow completes and a peak list is generated)
- The threshold must be configured manually in config.yaml, so inappropriate thresholds may over- or under-filter ions
- Requires pixel-resolved (imaging) peak-picked data; the metric cannot be computed on spectra without spatial information

## Evidence
