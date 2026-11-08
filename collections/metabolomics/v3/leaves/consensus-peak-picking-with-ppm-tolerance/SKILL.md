---
name: consensus-peak-picking-with-ppm-tolerance
description: Use when you have pixel-level DESI-MSI data along a tissue axis (e.g.
  maize root sections) and need a per-sample list of m/z signatures from untargeted
  spectra.
when_to_use_negative:
- Input is already a reduced peak/feature table rather than pixel-level imaging spectra
  — pseudobulk summation and ppm-based consensus calling would be redundant
- The tissue has no linear developmental axis to interpret, since this consensus-peak
  step is part of a pipeline (DIMPLE) designed for linear-gradient evaluation
- You need metabolite annotation/identification — pair the peak list with database
  search tools (METASPACE, Metlin Classic, LOTUS) rather than expecting it here
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DIMPLE
  role: Pipeline that performs pseudobulk summation, 2.5 ppm consensus peak extraction,
    background suppression and sparsity filtering on DESI-MSI data
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Imaging modality producing the pixel-level mass spectra that consensus peak
    picking is applied to
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:a3f871c37081ec760dd30d7ba63155e2ff59129ab7ce1986d0be9efd54304035
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# consensus-peak-picking-with-ppm-tolerance

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract consensus m/z peaks from DESI-MSI imaging runs by summing pixel intensities into a pseudobulk spectrum, then calling peaks within a 2.5 ppm mass tolerance and filtering by background and spatial sparsity. Used in DIMPLE to recover developmental mass signatures from maize root imaging data.

## When to use

Use when you have pixel-level DESI-MSI data along a tissue axis (e.g. maize root sections) and need a per-sample list of m/z signatures from untargeted spectra.

## When NOT to use

- Input is already a reduced peak/feature table rather than pixel-level imaging spectra — pseudobulk summation and ppm-based consensus calling would be redundant
- The tissue has no linear developmental axis to interpret, since this consensus-peak step is part of a pipeline (DIMPLE) designed for linear-gradient evaluation
- You need metabolite annotation/identification — pair the peak list with database search tools (METASPACE, Metlin Classic, LOTUS) rather than expecting it here

## Inputs

- DESI-MSI workspace files (Matlab .mat workspaces, e.g. B73-root11-workspace.mat) or raw cdf files for each replicate
- Optional reference peak list (e.g. 63 manually published peaks from Zhang et al. 2023) for recovery assessment

## Outputs

- Per-sample consensus m/z signature list (CSV)
- Pooled unique signature list across replicates (CSV)
- Recovery percentage against a reference peak list

## How to apply

For each DESI-MSI dataset (e.g. the Matlab workspaces B73-root11-workspace.mat, B73-root2-5-workspace.mat, B73-root-3-3-worskpace.mat in the DIMPLE-code repo), build a pseudobulk mass spectrum by summing the intensity of each m/z bin across all pixels. Extract consensus peaks from each pseudobulk spectrum using a 2.5 ppm mass tolerance to merge nearby m/z bins. Apply background suppression, retaining only peaks whose intensity exceeds 1.5-fold the background level, then apply a sparsity filter to remove m/z signatures present in only a small fraction of pixels. Compile the surviving m/z values as the per-root signature list and pool lists across replicates to count unique signatures; validate by computing recovery against a published reference peak list (e.g. 63 peaks from Zhang et al. 2023).

## Related tools

- **DIMPLE** (Pipeline that performs pseudobulk summation, 2.5 ppm consensus peak extraction, background suppression and sparsity filtering on DESI-MSI data) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Imaging modality producing the pixel-level mass spectra that consensus peak picking is applied to)

## Examples

```
load('B73-root11-workspace.mat'); % then run DIMPLE consensus peak extraction: sum intensities across pixels, call peaks at 2.5 ppm, apply 1.5x background and sparsity filters
```

## Evaluation signals

- Each replicate's signature list survives the 1.5-fold background threshold and sparsity filter; replicate counts average ~196 m/z signatures per root in the published B73 test case
- Pooling three B73 replicate lists yields ~301 unique signatures
- Recovery against the 63 manually published Zhang et al. 2023 peaks reaches ~97%, with the two missed peaks at very low abundance or sparse localization
- Merged peaks respect the 2.5 ppm mass tolerance (no duplicate signatures within tolerance)
- Per-root and pooled signature lists are saved as CSV files

## Limitations

- Peaks at very low abundance or with sparse localization may be missed even when present in a reference list (two of 63 Zhang et al. 2023 peaks were not recovered)
- DESI-MSI spatial resolution is only approximately 50-100 µm, limiting fine spatial attribution of signatures
- The pipeline is designed for tissues with a linear developmental axis (e.g. maize root); it was not intended for non-linear tissue geometry

## Evidence

- [methods] pseudobulk mass spectrum: [evidence span withheld — anchor status: not_found]
- [methods] 2.5 ppm tolerance and filters: [evidence span withheld — anchor status: not_found]
- [results] recovery metric: [evidence span withheld — anchor status: not_found]
