---
name: spatial-lipidomics-signal-quality-control
description: Use when you have pixel-wise peak-picked MALDI MSI spectra and need to
  remove ions that are noise or exogenous signals by scoring each ion's spatial coherence
  across pixels prior to normalisation.
when_to_use_negative:
- Input is already a normalised or segmented intensity matrix — spatial coherence
  filtering is applied on pixel-wise peak lists before intensity normalisation.
- Data are microscopy (IFM) images rather than MSI peak lists — msiFlow handles IFM
  via separate segmentation/registration workflows (msi_if_registration_flow, if_segmentation_flow).
- You need lipid annotation or biological interpretation — msiFlow explicitly does
  not include an annotation or interpretation step.
- The analysis requires spectral aggregation across pixels before peak detection —
  this workflow is designed around pixel-wise peak picking, not summed-spectrum picking.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: msiFlow
  role: Open-source workflow suite whose msi_preprocessing_flow performs pixel-wise
    peak picking and the spatial-coherence peak filtering step on raw MALDI MSI data
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine automating and parallelising the pre-processing and peak filtering
    steps for high-throughput analyses
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: MALDI-2
  role: Instrument modality (matrix-assisted laser desorption ionisation with laser-induced
    postionisation) generating the raw lipidomic MSI data the filter is applied to
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:22326bf259c10d3d2f4d8f2bca4898adb84fa0b5b8a5c06a2764401e7e7232dd
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
  - MALDI-2
  - Snakemake
  - msiFlow
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# spatial-lipidomics-signal-quality-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Signal quality control for MALDI MSI lipidomics that goes beyond standard peak picking by computing a spatial coherence metric per ion and removing non-informative or exogenous signals whose intensity distributions lack spatial structure before intensity normalisation.

## When to use

Use when you have pixel-wise peak-picked MALDI MSI spectra and need to remove ions that are noise or exogenous signals by scoring each ion's spatial coherence across pixels prior to normalisation.

## When NOT to use

- Input is already a normalised or segmented intensity matrix — spatial coherence filtering is applied on pixel-wise peak lists before intensity normalisation.
- Data are microscopy (IFM) images rather than MSI peak lists — msiFlow handles IFM via separate segmentation/registration workflows (msi_if_registration_flow, if_segmentation_flow).
- You need lipid annotation or biological interpretation — msiFlow explicitly does not include an annotation or interpretation step.
- The analysis requires spectral aggregation across pixels before peak detection — this workflow is designed around pixel-wise peak picking, not summed-spectrum picking.

## Inputs

- pixel-wise peak-picked MALDI MSI spectra (peak list derived from raw MALDI MSI data, e.g. t-MALDI-2 high-resolution lipidomic imaging data)
- workflow config.yaml specifying the spatial coherence threshold

## Outputs

- filtered_peak_list.csv containing retained ions (m/z and intensities) that pass the spatial coherence threshold

## How to apply

Run the msiFlow pre-processing environment (Snakemake `msi_preprocessing_flow`), which generates a peak list from raw MALDI MSI data (e.g. the t-MALDI-2 example data from Zenodo deposit 10.5281/zenodo.11913042). The workflow performs pixel-wise peak picking and then, rather than retaining every picked peak, computes the spatial coherence metric for each ion from its intensity distribution across pixels. Ions whose spatial coherence score falls below the coherence threshold configured in the workflow's config.yaml are removed as non-informative. The surviving ions are assembled into a filtered peak table written to filtered_peak_list.csv, which is produced immediately before the intensity normalisation step. The rationale is that genuinely biological lipid signals should show spatially structured intensity patterns across the tissue, so spatial coherence serves as an informativeness filter on top of standard peak picking.

## Related tools

- **msiFlow** (Open-source workflow suite whose msi_preprocessing_flow performs pixel-wise peak picking and the spatial-coherence peak filtering step on raw MALDI MSI data) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine automating and parallelising the pre-processing and peak filtering steps for high-throughput analyses) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **MALDI-2** (Instrument modality (matrix-assisted laser desorption ionisation with laser-induced postionisation) generating the raw lipidomic MSI data the filter is applied to)

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git
```

## Evaluation signals

- filtered_peak_list.csv exists after the workflow completes and is a readable tabular file with ion m/z and intensity columns.
- Every ion retained in filtered_peak_list.csv has a spatial coherence score at or above the threshold configured in config.yaml.
- The filtered peak list is visibly smaller than the raw picked peak list, demonstrating removal of non-informative or exogenous signals.
- The filtering step occurs immediately before intensity normalisation in the executed pipeline order, matching the msi_preprocessing_flow design.
- The workflow runs reproducibly within the msiFlow Docker image/Snakemake environment on the Zenodo example dataset (10.5281/zenodo.11913042) or the bundled demo dataset.

## Limitations

- The article does not state a default numeric value for the spatial coherence threshold; it must be configured per dataset in config.yaml.
- Filtering is qualitative in its acceptance criteria — retention is defined by passing the spatial coherence threshold, not by annotation or biological validation.
- msiFlow does not include an annotation or interpretation step, so filtered peaks still require downstream mass matching (e.g. to LipidMaps) for lipid identity.
- Cell-specificity of filtered lipid signals is limited by the use of consecutive tissue sections and lateral diffusion of lipids during MSI sample preparation.

## Evidence
