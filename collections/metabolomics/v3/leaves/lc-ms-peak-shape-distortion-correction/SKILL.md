---
name: lc-ms-peak-shape-distortion-correction
description: Use when ncGTW realignment has distorted peak shapes (broadened or narrowed
  peaks) in XCMS-flagged misaligned features, and corrected warping functions plus
  relocated peak positions are needed to return results to XCMS.
when_to_use_negative:
- When only misalignment detection or profile realignment is needed (no peak-shape
  distortion to correct) — use the detection/realignment modules instead
- When XCMS did not flag the features as misaligned and the profiles are already well-aligned,
  since the correction is targeted at realigned ncGTW features
- When no raw LC-MS profiles or XCMS peak start/end boundaries are available — the
  module needs apex boundaries and peak widths, not just a feature table
tools:
- name: ncGTW
  role: Provides the post-processing module that corrects peak shape distortion after
    realignment (adjustRT and ncGTWpeaks)
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Source of flagged misaligned features, peak start/end points and peak widths;
    receives the corrected warping functions and relocated peaks for grouping and
    peak-filling
- name: R
  role: Implementation language; ncGTW runs as an R plug-in to XCMS
  repo: https://github.com/ChiungTingWu/ncGTW
- name: Bioconductor
  role: Distribution channel for installing the ncGTW R package
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:ab8c0b032b3401a4e5c747d463263ebc5b89d63e903ad37a6b8f8e7d3159244a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - Bioconductor
  - R
  - XCMS
  - ncGTW
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lc-ms-peak-shape-distortion-correction

## Summary

The ncGTW post-processing module that converts feature realignment results into corrected RT warping functions and relocated peak apex/positions, preventing DTW-induced peak broadening/narrowing so realigned profiles can be fed back into XCMS for grouping and peak-filling.

## When to use

Use when ncGTW realignment has distorted peak shapes (broadened or narrowed peaks) in XCMS-flagged misaligned features, and corrected warping functions plus relocated peak positions are needed to return results to XCMS.

## When NOT to use

- When only misalignment detection or profile realignment is needed (no peak-shape distortion to correct) — use the detection/realignment modules instead
- When XCMS did not flag the features as misaligned and the profiles are already well-aligned, since the correction is targeted at realigned ncGTW features
- When no raw LC-MS profiles or XCMS peak start/end boundaries are available — the module needs apex boundaries and peak widths, not just a feature table

## Inputs

- ncGTW realigned feature profiles (output of the realignment step)
- XCMS peak start/end points and peak widths for each flagged feature
- xcmsLargeWin XCMSnExp object with corrected RT (rt$corrected) and feature grouping

## Outputs

- New RT warping functions replacing xcmsLargeWin@rt$corrected
- ncGTWpeaks: relocated peak positions with apexes aligned at the new RT points
- Per-feature corrected intensity table across samples (for downstream XCMS grouping and peak-filling)

## How to apply

Load the ncGTW realigned feature profiles (output of the realignment step) together with the XCMS-provided peak start/end points and peak widths for each flagged feature. For each feature and sample, restrict the warping function to the apex point within the peak's start-end boundaries and align only the apex; then shift all remaining points inside each peak boundary rigidly by the same apex displacement, which prevents the many-to-one/one-to-many mappings that DTW-based warping would otherwise introduce. For samples with missing peaks undetected by XCMS, locate the apex on the virtual reference aggregated from other samples, assign the corresponding warping-function point as the apex, and align the whole missing peak using the median of the peak widths. Assemble the corrected profiles into a per-feature intensity table across samples, generating new RT warping functions that replace xcmsLargeWin@rt$corrected and ncGTWpeaks with relocated apex positions for downstream XCMS grouping and peak-filling.

## Related tools

- **ncGTW** (Provides the post-processing module that corrects peak shape distortion after realignment (adjustRT and ncGTWpeaks)) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Source of flagged misaligned features, peak start/end points and peak widths; receives the corrected warping functions and relocated peaks for grouping and peak-filling)
- **R** (Implementation language; ncGTW runs as an R plug-in to XCMS) — https://github.com/ChiungTingWu/ncGTW
- **Bioconductor** (Distribution channel for installing the ncGTW R package)

## Examples

```
devtools::install_github("ChiungTingWu/ncGTW")
```

## Evaluation signals

- Within each corrected feature, the apices remain co-aligned across samples (mirrors the Fig. 5c-to-5d correction criterion)
- Peak shape (width and height profile) of each corrected peak is preserved relative to the pre-correction realigned profiles
- No many-to-one/one-to-many RT mappings appear within peak boundaries (shift inside each peak is rigid, equal to the apex displacement)
- For samples with missing peaks, the assigned apex on the virtual reference uses the median of the peak widths, and the whole peak is aligned consistently
- Downstream XCMS grouping and peak-filling run successfully on the realigned/corrected data (adjustRT output replaces rt$corrected without schema errors)

## Limitations

- Peak distortion is an inherent risk of ncGTW's alignment; parameter settings must be adjusted or optimized to reduce the likelihood of distortion, and defaults are not exhaustively documented in the paper
- False positives from the misalignment detection module (e.g. higher false positive rate observed in the Rotterdam dataset) are accepted and only screened out in a later post-alignment step via correlation coefficient and overlapping area scores
- The paper does not provide exact runtime/computational cost metrics for this module on large-scale datasets, nor XCMS/ncGTW version numbers

## Evidence

- [methods] Peak shape distortion is then efficiently corrected by the post-processing module of ncGTW package.: [evidence span withheld — anchor status: not_found]
