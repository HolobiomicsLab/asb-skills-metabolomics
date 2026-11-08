---
name: signature-list-compilation-and-pooling
description: Use when you have pixel-level DESI-MSI data over a tissue axis (e.g.,
  .cdf or Matlab workspace files) and need per-sample m/z signature lists plus a pooled
  replicate-level unique signature set.
when_to_use_negative:
- Input is already a curated feature/peak table or annotation result — this skill
  operates on raw pixel-level MSI spectra via pseudobulking
- Data are not acquired over a linear spatial axis; the DIMPLE framing (and the pseudobulk
  + sparsity filters) is designed for tissues like the developing maize root along
  a longitudinal developmental gradient
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: DIMPLE
  role: Pipeline that processes mass signatures along linear gradients; implements
    consensus peak extraction, background suppression, and sparsity filtering to compile
    signature lists
  repo: https://github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: 'Data source: desorption electrospray ionization mass spectrometry imaging
    of maize root sections provides the pixel-level mass spectra that pseudobulking
    aggregates'
- name: batchcdfread
  role: Matlab function (developed by Yifan Meng, Zare lab, Stanford) for reading
    the .cdf mass spec files into the pipeline
  repo: https://github.com/dickinsonlab/DIMPLE-code
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:43a543be6efb9a265c44278b7250a5b0a7a8f3d3a4891337e83044e19f212ded
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/dickinsonlab/DIMPLE-code
  repo_url: https://github.com/dickinsonlab/DIMPLE-code
  tools:
  - DESI-MSI
  - DIMPLE
  - batchcdfread
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# signature-list-compilation-and-pooling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compile per-sample lists of m/z mass signatures from DESI-MSI pseudobulk spectra and pool replicates into a unique signature set, as done with DIMPLE on three B73 maize root datasets (2.5 ppm consensus peak extraction, 1.5-fold background suppression, sparsity filtering).

## When to use

Use when you have pixel-level DESI-MSI data over a tissue axis (e.g., .cdf or Matlab workspace files) and need per-sample m/z signature lists plus a pooled replicate-level unique signature set.

## When NOT to use

- Input is already a curated feature/peak table or annotation result — this skill operates on raw pixel-level MSI spectra via pseudobulking
- Data are not acquired over a linear spatial axis; the DIMPLE framing (and the pseudobulk + sparsity filters) is designed for tissues like the developing maize root along a longitudinal developmental gradient

## Inputs

- DESI-MSI pixel-level datasets as NetCDF (.cdf) files or Matlab workspace .mat files (e.g., B73-root11-workspace.mat)
- Mass tolerance parameter (2.5 ppm)
- Background suppression threshold (1.5-fold background intensity)
- Optional: published reference peak list for recovery comparison (e.g., 63 peaks from Zhang et al. 2023)

## Outputs

- Per-root/per-replicate mass signature list (m/z values), saved as CSV
- Pooled unique signature list across replicates (301 unique m/z for three B73 roots), saved as CSV
- Recovery percentage against a reference peak set (97% of 63 Zhang et al. 2023 peaks)

## How to apply

Load each replicate's DESI-MSI dataset (e.g., B73-root11, B73-root2-5, B73-root-3-3 Matlab workspaces from the DIMPLE-code repository) and build a pseudobulk mass spectrum by summing each m/z bin's intensity across all pixels. Extract consensus peaks from each pseudobulk spectrum with a 2.5 ppm mass tolerance, then apply background suppression retaining only peaks whose intensity exceeds 1.5-fold the background level. Follow with sparsity filtering to remove m/z signatures present in only a small fraction of pixels. Compile the surviving m/z values as the per-root signature list, pool the three lists to count unique signatures, and optionally compare against a published reference peak set (63 peaks from Zhang et al. 2023) to compute recovery percentage. Rationale: pseudobulking stabilizes peak detection on noisy per-pixel spectra, and background/sparsity filters remove artifacts that do not represent genuine spatially localized metabolites.

## Related tools

- **DIMPLE** (Pipeline that processes mass signatures along linear gradients; implements consensus peak extraction, background suppression, and sparsity filtering to compile signature lists) — https://github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Data source: desorption electrospray ionization mass spectrometry imaging of maize root sections provides the pixel-level mass spectra that pseudobulking aggregates)
- **batchcdfread** (Matlab function (developed by Yifan Meng, Zare lab, Stanford) for reading the .cdf mass spec files into the pipeline) — https://github.com/dickinsonlab/DIMPLE-code

## Examples

```
matlab -batch "load('B73-root11-workspace.mat')"  % then build pseudobulk spectrum, extract 2.5 ppm consensus peaks, apply 1.5-fold background and sparsity filters
```

## Evaluation signals

- Average per-root signature count falls near 196 m/z signatures per root for the three B73 replicates
- Pooled unique signature count matches ~301 signatures across three B73 root replicates
- Recovery against Zhang et al. 2023 reference reaches ~97% of the 63 published peaks; the two missed signatures are expected to be very low abundance or sparsely localized
- Retained peaks pass both thresholds: intensity > 1.5-fold background and non-sparse pixel localization
- Per-root and pooled signature lists are written as CSV files and are mutually consistent (pooled list is the union of per-root lists)

## Limitations

- Signatures at very low abundance or with sparse localization may be missed by the background suppression and sparsity filters (two of 63 Zhang et al. 2023 peaks were not recovered)
- Each pixel contains a full mass spectrum with hundreds to thousands of peaks, so untargeted analysis of the full data volume is challenging; most metabolites remain unannotated
- Filter thresholds (2.5 ppm tolerance, 1.5-fold background) were calibrated for the B73/Oaxacan Green maize root DESI-MSI datasets and may need re-tuning for other tissues or instruments

## Evidence

- [methods] Extract consensus peaks from each pseudobulk spectrum with a 2.5 ppm mass tolerance: [evidence span withheld — anchor status: not_found]
- [methods] Background suppression and sparsity filtering thresholds: [evidence span withheld — anchor status: not_found]
- [methods] Expected verification metrics for the compiled signature lists: [evidence span withheld — anchor status: not_found]
