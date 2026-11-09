---
name: pseudobulk-spectrum-construction
description: Use when analyzing MSI data with hundreds-to-thousands of peaks per pixel,
  where untargeted analysis of the full pixel-level mass spectrum is intractable and
  a per-pixel pooled spectrum is needed.
when_to_use_negative:
- Input is already a per-sample pooled spectrum or a peak table — summation would
  be redundant.
- Targeted analysis of a known m/z where a per-pixel spatial map is required — summation
  destroys the spatial dimension needed for localization.
- Data from a non-linear developmental axis where linear gradient assumptions of DIMPLE
  do not apply.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DESI-MSI
  role: Source instrument/data type that produces the pixel-level mass spectra being
    pooled
- name: DIMPLE
  role: Pipeline whose first aggregation step produces the pseudobulk spectrum before
    consensus-peak extraction
  repo: github.com/dickinsonlab/DIMPLE-code
- name: batchcdfread
  role: Function used to read the deposited NetCDF (.cdf) DESI-MSI files into memory
    for summation
  repo: github.com/dickinsonlab/DIMPLE-code
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:4adeec58fa0f7b1f18c5f17a878f8e9435a945197b67d996b9f442e2e97cdc49
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/dickinsonlab/DIMPLE-code
  tools:
  - DESI-MSI
  - DIMPLE
  - batchcdfread
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pseudobulk-spectrum-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sum DESI-MSI pixel intensities across a spatial dataset into a single pseudobulk mass spectrum per root, providing the m/z intensity profile from which consensus peaks are extracted in the DIMPLE pipeline.

## When to use

Use when analyzing MSI data with hundreds-to-thousands of peaks per pixel, where untargeted analysis of the full pixel-level mass spectrum is intractable and a per-pixel pooled spectrum is needed.

## When NOT to use

- Input is already a per-sample pooled spectrum or a peak table — summation would be redundant.
- Targeted analysis of a known m/z where a per-pixel spatial map is required — summation destroys the spatial dimension needed for localization.
- Data from a non-linear developmental axis where linear gradient assumptions of DIMPLE do not apply.

## Inputs

- DESI-MSI dataset of a root (e.g. NetCDF .cdf file or MATLAB workspace .mat) with pixel-level mass spectra

## Outputs

- One pseudobulk mass spectrum per root (m/z bin intensities pooled across all pixels)
- Per-root mass-signature lists and pooled unique signature list (CSV files)

## How to apply

Sum the intensity of each m/z bin across all pixels of the input DESI-MSI dataset to produce a single pooled mass spectrum per sample. Per-root spectra are computed from each of the three B73 maize root datasets (e.g. B73-root11, B73-root2-5, B73-root-3-3) loaded from the deposited MATLAB workspaces in the DIMPLE-code repository; the batchcdfread function is used to read the cdf files. The resulting pseudobulk spectrum is the substrate for the downstream consensus-peak extraction step, which uses a 2.5 ppm mass tolerance and 1.5-fold background suppression with sparsity filtering, so the summation must preserve m/z bin identity and intensity fidelity. Save the per-root signature lists and pooled unique signature list as CSV files for comparison against published peaks (e.g. Zhang et al. 2023, 63 peaks).

## Related tools

- **DESI-MSI** (Source instrument/data type that produces the pixel-level mass spectra being pooled)
- **DIMPLE** (Pipeline whose first aggregation step produces the pseudobulk spectrum before consensus-peak extraction) — github.com/dickinsonlab/DIMPLE-code
- **batchcdfread** (Function used to read the deposited NetCDF (.cdf) DESI-MSI files into memory for summation) — github.com/dickinsonlab/DIMPLE-code

## Evaluation signals

- Per-root spectrum must reproduce summed (not averaged) intensities across all pixels of the root dataset.
- Three B73 root pseudobulk spectra must be generated, one from each of B73-root11, B73-root2-5, and B73-root-3-3.
- Downstream consensus-peak extraction should yield ~196 m/z signatures per root on average and 301 unique signatures across three replicates.
- 97% recovery of the 63 manually published peaks from Zhang et al. 2023 when pooled unique signatures are compared, indicating the summation step preserved usable intensity fidelity.
- Per-root signature lists and pooled unique signature list must be exported as CSV files.

## Limitations

- Pseudobulk summation masks the spatial localization of each m/z signal; spatial information is only recovered later via cluster/pattern analysis in DIMPLE, not from the pooled spectrum.
- Recovery of low-abundance or sparsely-localized signatures may be reduced — two of the 63 Zhang et al. 2023 peaks were missed by the downstream pipeline at very low abundance or sparse localization.
- Requires deposited .cdf or MATLAB workspace data with consistent m/z binning; any bin mismatch across replicates would preclude direct pooling of per-root spectra.

## Evidence
