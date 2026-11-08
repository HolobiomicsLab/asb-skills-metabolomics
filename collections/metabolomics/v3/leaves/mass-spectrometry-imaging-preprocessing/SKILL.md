---
name: mass-spectrometry-imaging-preprocessing
description: Use when raw DESI-MSI root-imaging data (NetCDF .cdf or MATLAB workspace
  .mat files) must be converted into per-sample mass signature lists before gradient
  clustering or cross-replicate comparison.
when_to_use_negative:
- Data are already an annotated feature table or assigned metabolite identities —
  this skill stops at raw consensus m/z list extraction, not downstream annotation
  or METASPACE/Metlin/LOTUS lookup
- Sample is not arranged along a linear developmental axis (e.g. 2D tissue maps without
  a longitudinal root gradient) — DIMPLE's pseudobulk and linear-gradient assumptions
  do not apply
- Spatial resolution or pixel density is too coarse to support per-pixel binning (e.g.
  low-resolution MALDI or SIMS data where the ~50-100 µm DESI-MSI assumption breaks
  down)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DESI-MSI
  role: Source imaging modality producing the per-pixel mass spectra that are binned
    and pseudobulked per root
- name: DIMPLE
  role: Computational pipeline that implements pseudobulk spectrum building, 2.5 ppm
    consensus peak extraction, background suppression, and sparsity filtering
  repo: github.com/dickinsonlab/DIMPLE-code
- name: batchcdfread
  role: MATLAB function used to read the raw .cdf mass-spectrometry-imaging files
    into MATLAB before preprocessing
  repo: github.com/dickinsonlab/DIMPLE-code
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:d215500c189853a6ac0c0c851852894bbd7d9c31d4285755691d0822f50fd9ba
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# mass-spectrometry-imaging-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocesses DESI-MSI workspaces of maize root sections into per-root pseudobulk mass spectra and extracts consensus m/z signatures under DIMPLE's 2.5 ppm tolerance, background-enrichment, and sparsity filters, supporting developmental-gradient analysis as in Sama et al. 2025.

## When to use

Use when raw DESI-MSI root-imaging data (NetCDF .cdf or MATLAB workspace .mat files) must be converted into per-sample mass signature lists before gradient clustering or cross-replicate comparison.

## When NOT to use

- Data are already an annotated feature table or assigned metabolite identities — this skill stops at raw consensus m/z list extraction, not downstream annotation or METASPACE/Metlin/LOTUS lookup
- Sample is not arranged along a linear developmental axis (e.g. 2D tissue maps without a longitudinal root gradient) — DIMPLE's pseudobulk and linear-gradient assumptions do not apply
- Spatial resolution or pixel density is too coarse to support per-pixel binning (e.g. low-resolution MALDI or SIMS data where the ~50-100 µm DESI-MSI assumption breaks down)

## Inputs

- DESI-MSI NetCDF (.cdf) files of maize root sections, e.g. B73-root11, B73-root2-5, B73-root-3-3, OG1, OG5, OG2-P11-0
- Matlab workspaces (e.g. B73-root11-workspace.mat, OG1-workspace-2.mat) accompanying each .cdf file
- batchcdfread function (developed by Yifan Meng, Zare lab, Stanford) for reading the .cdf mass spec files

## Outputs

- Per-root mass signature list (surviving m/z values after 2.5 ppm consensus, 1.5× background, and sparsity filtering)
- Pooled unique signature list across the three B73 replicates (e.g. 301 unique m/z signatures)
- Recovery metrics vs. a published reference peak set (e.g. 97% recovery of the 63 Zhang et al. 2023 B73 peaks)
- Parameter-sensitivity metrics table (e.g. parameter_sensitivity_metrics.csv) comparing 2.5 ppm/1.5× defaults to 5 ppm and 1.1× relaxed settings

## How to apply

Load each deposited B73 root DESI-MSI workspace (e.g., B73-root11, B73-root2-5, B73-root-3-3 from the DIMPLE-code repository, including the batchcdfread function by Yifan Meng in Dr. Richard N. Zare's lab) and, for each dataset, build a pseudobulk mass spectrum by summing the intensity of each m/z bin across all pixels. Extract consensus peaks from each pseudobulk spectrum with a 2.5 ppm mass tolerance — this is the default DIMPLE parameter; raising it to 5 ppm increases identified signatures by ~14% (196→223 per root). Suppress background by retaining only peaks whose intensity exceeds 1.5-fold the background level (lowering this to 1.1 adds ~3% peaks), then apply a sparsity filter that removes m/z signatures present in only a small fraction of pixels. Compile the surviving m/z values as the per-root mass signature list and pool lists across replicates to count unique signatures. The two-step filter (background then sparsity) is the decision point: it must remove low-intensity and sparsely localized peaks while keeping the ~196 per-root average observed on B73.

## Related tools

- **DESI-MSI** (Source imaging modality producing the per-pixel mass spectra that are binned and pseudobulked per root)
- **DIMPLE** (Computational pipeline that implements pseudobulk spectrum building, 2.5 ppm consensus peak extraction, background suppression, and sparsity filtering) — github.com/dickinsonlab/DIMPLE-code
- **batchcdfread** (MATLAB function used to read the raw .cdf mass-spectrometry-imaging files into MATLAB before preprocessing) — github.com/dickinsonlab/DIMPLE-code

## Examples

```
run DIMPLE preprocessing on B73-root11-workspace.mat with ppm=2.5, bg_threshold=1.5, sparsity_filter=on; save B73-root11_signature_list.csv
```

## Evaluation signals

- Per-root signature count averages ~196 m/z peaks across the three B73 replicates under default 2.5 ppm / 1.5× background / sparsity parameters
- Pooled unique signatures across three B73 replicates total 301
- 97% recovery of the 63 manually published peaks from Zhang et al. 2023; the two missed signatures are at very low abundance or sparse localization
- Relaxing ppm tolerance from 2.5 to 5 yields ~223 peaks per root (+14%); lowering background threshold from 1.5 to 1.1 yields ~199 peaks per root (+3%)
- Each per-root output CSV contains a nonempty list of m/z values within expected mass range and no duplicates after 2.5 ppm clustering

## Limitations

- Using the relaxed 5 ppm or 1.1× background thresholds inflates peak counts (+14% and +3% respectively), so cross-study comparisons must fix one parameter set
- Two of the 63 Zhang et al. 2023 reference peaks are not recovered by the pipeline because they are very low abundance or sparsely localized, so near-perfect recovery does not mean perfect recovery
- The majority of detected mass signatures remain unannotated; this preprocessing step does not assign metabolite identities
- This is the DIMPLE version as described in Sama et al. 2025; it was designed for linear root developmental gradients and may not generalize to other geometries

## Evidence

- [methods] Building pseudobulk spectra by summing m/z bins across pixels: [evidence span withheld — anchor status: not_found]
- [methods] Consensus peak extraction tolerance: [evidence span withheld — anchor status: not_found]
- [methods] Background suppression threshold: [evidence span withheld — anchor status: not_found]
- [methods] Sparsity filtering removes low-coverage m/z: [evidence span withheld — anchor status: not_found]
