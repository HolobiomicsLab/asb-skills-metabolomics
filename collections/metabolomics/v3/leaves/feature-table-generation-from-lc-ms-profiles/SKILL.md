---
name: feature-table-generation-from-lc-ms-profiles
description: Use when you have ncGTW realignment output for misaligned XCMS features
  plus XCMS peak start/end points and widths, and need to convert warping functions
  back into corrected retention times and a per-feature intensity table across samples.
when_to_use_negative:
- The input is an already-corrected feature table with no raw realigned profiles —
  there are no warping functions to convert and no apex/width boundaries to rigidly
  shift.
- Features were not flagged as misaligned by the ncGTW misalignment detection module
  — XCMS's single warping-function alignment is sufficient and reprocessing adds distortion
  risk.
- The goal is detection or realignment of misaligned features itself (the upstream
  ncGTW alignment step), not converting realignment results back into an XCMS feature
  table.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ncGTW
  role: Provides adjustRT() and ncGTWpeaks and the realigned feature profiles from
    which the corrected RT warping functions, relocated peak positions, and feature
    intensity table are built
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Supplies the peak start/end points, peak widths, and xcmsLargeWin@rt$corrected
    slot that the corrected warping functions replace; receives the corrected feature
    table back for grouping and peak-filling
- name: R
  role: Implementation language; ncGTW runs as an R package plug-in installed from
    Bioconductor or GitHub
  repo: https://github.com/ChiungTingWu/ncGTW
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:1bac4d4ba1b854852125b1e53844769003dd3aebf6c0ba28c73d56b4ae02ccf3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - R
  - XCMS
  - ncGTW
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# feature-table-generation-from-lc-ms-profiles

## Summary

Generates an XCMS-compatible, distortion-corrected feature intensity table from ncGTW realigned LC-MS profiles by converting realignment warping functions into corrected RT coordinates and relocating peaks, so that XCMS can proceed with grouping and peak-filling on corrected data.

## When to use

Use when you have ncGTW realignment output for misaligned XCMS features plus XCMS peak start/end points and widths, and need to convert warping functions back into corrected retention times and a per-feature intensity table across samples.

## When NOT to use

- The input is an already-corrected feature table with no raw realigned profiles — there are no warping functions to convert and no apex/width boundaries to rigidly shift.
- Features were not flagged as misaligned by the ncGTW misalignment detection module — XCMS's single warping-function alignment is sufficient and reprocessing adds distortion risk.
- The goal is detection or realignment of misaligned features itself (the upstream ncGTW alignment step), not converting realignment results back into an XCMS feature table.

## Inputs

- ncGTW realigned feature profiles (output of the ncGTW realignment step)
- XCMS peak start/end points and peak widths for each flagged feature
- xcmsLargeWin object with XCMS peak detection, RT alignment, and peak grouping already applied

## Outputs

- Corrected RT warping functions replacing xcmsLargeWin@rt$corrected
- Relocated peak positions (ncGTWpeaks) on the corrected RT coordinates
- Per-feature corrected intensity table across samples, compatible with downstream XCMS grouping and peak-filling

## How to apply

Load the ncGTW realigned feature profiles together with the XCMS-provided peak start/end points and peak widths for each flagged feature. For each feature and sample, restrict the warping function to the apex point within the peak's start-end boundaries so only the apex is aligned, then shift all remaining points inside each peak boundary rigidly by the same apex displacement — this prevents the many-to-one/one-to-many mappings that DTW-based warping would otherwise introduce to broaden or narrow peaks. For samples where XCMS did not detect the peak, locate the apex on the virtual reference aggregated from the other samples, assign the corresponding warping-function point as the apex, and align the whole missing peak using the median of the peak widths. Apply adjustRT() to the realignment output to generate new RT warping functions that replace xcmsLargeWin@rt$corrected, use ncGTWpeaks to relocate peak positions onto the new RT points, and assemble the corrected profiles into a per-feature intensity table across samples that is handed back to XCMS for downstream grouping and peak-filling.

## Related tools

- **ncGTW** (Provides adjustRT() and ncGTWpeaks and the realigned feature profiles from which the corrected RT warping functions, relocated peak positions, and feature intensity table are built) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Supplies the peak start/end points, peak widths, and xcmsLargeWin@rt$corrected slot that the corrected warping functions replace; receives the corrected feature table back for grouping and peak-filling)
- **R** (Implementation language; ncGTW runs as an R package plug-in installed from Bioconductor or GitHub) — https://github.com/ChiungTingWu/ncGTW

## Examples

```
BiocManager::install("ncGTW")
```

## Evaluation signals

- Within each corrected feature, the apex RTs remain co-aligned across samples after correction.
- Peak shape (width and height profile) of each peak is preserved relative to the pre-correction realigned profiles — no broadening or narrowing from many-to-one/one-to-many warping mappings.
- xcmsLargeWin@rt$corrected has been replaced by the new RT warping functions produced by adjustRT().
- For samples with peaks undetected by XCMS, the missing peak is aligned using the median of the peak widths against the virtual reference aggregated from other samples.
- The resulting per-feature intensity table is accepted by XCMS for downstream grouping and peak-filling without format errors.

## Limitations

- Peak distortion can still occur if ncGTW parameter settings are not adjusted; parameter settings 'can be adjusted or optimized to reducing the likelihood of peak distortion' and are not exhaustively listed in the paper — users must consult the package defaults.
- The post-realignment stage also needs false-positive screening: potentially misaligned features (e.g., 57 flagged in the study) must be filtered using correlation coefficient and overlapping-area performance scores before/while building the corrected table.
- The skill only corrects features for which a realignment was produced; features that remain uncorrectable are the ones existing methods would discard.

## Evidence

- [methods] Peak shape distortion is then efficiently corrected by the post-processing module of ncGTW package.: [evidence span withheld — anchor status: not_found]
