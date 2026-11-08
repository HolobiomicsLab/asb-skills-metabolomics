---
name: dtw-warping-postprocessing
description: Use when XCMS-flagged misaligned features have been realigned by ncGTW
  and the realigned profiles must be converted back into XCMS RT warping functions
  and repositioned peaks for downstream grouping and peak-filling.
when_to_use_negative:
- Features that were not flagged by the ncGTW misalignment detection module and are
  already well aligned — the apex-shift correction targets flagged, distorted features
  only
- Raw LC-MS profiles that have not yet undergone XCMS peak detection, RT alignment,
  and peak grouping — the skill consumes XCMS peak boundaries and warping functions,
  not raw scan data
- Cases where a single sample-level warping function is adequate and compound-specific
  RT drift is not an issue
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ncGTW
  role: R/Bioconductor plug-in providing adjustRT() and ncGTWpeaks to build the corrected
    warping functions and relocate peaks
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Supplies the xcmsLargeWin object, peak start/end boundaries and peak widths;
    consumes the corrected output for grouping and peak-filling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:4eb1660782490477f8bc15d178c4abfa874c8460894e7bf13fe2771e8d49291c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - R
  - XCMS
  - ncGTW
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# dtw-warping-postprocessing

## Summary

Post-processing module of ncGTW that converts feature realignment results into XCMS-compatible RT warping functions and relocated peak positions, correcting the peak shape distortion that DTW-style warping otherwise introduces.

## When to use

Use when XCMS-flagged misaligned features have been realigned by ncGTW and the realigned profiles must be converted back into XCMS RT warping functions and repositioned peaks for downstream grouping and peak-filling.

## When NOT to use

- Features that were not flagged by the ncGTW misalignment detection module and are already well aligned — the apex-shift correction targets flagged, distorted features only
- Raw LC-MS profiles that have not yet undergone XCMS peak detection, RT alignment, and peak grouping — the skill consumes XCMS peak boundaries and warping functions, not raw scan data
- Cases where a single sample-level warping function is adequate and compound-specific RT drift is not an issue

## Inputs

- ncGTW realigned feature profiles for misaligned feature groups (output of the ncGTW realignment step)
- XCMS object (xcmsLargeWin) with @rt$corrected RT warping functions
- XCMS-provided peak start/end points and peak widths for each flagged feature and sample
- Correlation coefficient and overlapping area scores from the post-alignment step

## Outputs

- New per-sample RT warping functions replacing xcmsLargeWin@rt$corrected
- ncGTWpeaks: relocated peak positions at the new RT points
- Per-feature intensity table across samples with corrected (distortion-free) peak profiles

## How to apply

Load the ncGTW realigned feature profiles together with the XCMS peak start/end points and peak widths for each flagged feature. Apply adjustRT() to the ncGTW realignment output to generate new RT warping functions that replace xcmsLargeWin@rt$corrected. For each feature and sample, restrict the warping function to the apex within the peak's start-end boundaries, then rigidly shift all remaining points inside the peak boundary by the same displacement as the apex — this prevents the many-to-one/one-to-many mappings of DTW-based warping that broaden or narrow peaks. For samples where XCMS missed the peak, locate the apex on the virtual reference aggregated from the other samples, assign the corresponding warping-function point as the apex, and align the whole peak using the median of the peak widths. Use ncGTWpeaks to relocate peaks to the new RT points and assemble a per-feature intensity table across samples, then screen true positives via the correlation coefficient and overlapping area scores before sending results back to XCMS.

## Related tools

- **ncGTW** (R/Bioconductor plug-in providing adjustRT() and ncGTWpeaks to build the corrected warping functions and relocate peaks) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Supplies the xcmsLargeWin object, peak start/end boundaries and peak widths; consumes the corrected output for grouping and peak-filling)

## Evaluation signals

- Within each corrected feature, apex RTs are co-aligned across samples (the Fig. 5c-to-5d correction criterion)
- Peak shape (width/height profile) is preserved relative to the pre-correction realigned profiles
- The warping function restricted to each peak boundary is rigid (single apex displacement), with no many-to-one/one-to-many mappings that would broaden or narrow peaks
- True positives survive screening by the correlation coefficient and overlapping area scores; false positives are filtered out at this later stage
- Downstream XCMS grouping and peak-filling runs successfully on the corrected data

## Limitations

- DTW-based warping inherently produces many-to-one/one-to-many mappings that can broaden or narrow peaks; this module mitigates but does not eliminate dependence on the upstream warping quality
- ncGTW has a higher-than-theoretical false positive rate (e.g., in the Rotterdam dataset it exceeded the theoretical 0.05 threshold), so this post-processing stage must also screen out false positives
- Parameter settings must be adjusted or optimized to reduce the likelihood of peak distortion; defaults are not exhaustively documented in the paper

## Evidence

- [methods] Peak shape distortion is then efficiently corrected by the post-processing module of ncGTW package.: [evidence span withheld — anchor status: not_found]
- [methods] Post-alignment false-positive screening via two scores.: "via a post-alignment step we use these two performance scores to screen out true positives for further analysis."
- [readme] ncGTW as XCMS plug-in: realigned data returned for grouping and peak-filling.: "That is, ncGTW can detect the misaligned feature groups from XCMS and realign them. After that, XCMS can use the realigned data from
XCMS for more accurate grouping and peak-filling."
- [discussion] False positives accepted then screened out at a later stage.: "we have opted to first ‘accept’ these false positives and then screen them out at
later stage."
- [discussion] Parameter tuning reduces peak distortion likelihood.: "parameter settings in ncGTW can be adjusted or optimized to reducing the likelihood of peak
distortion."
