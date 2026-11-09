---
name: profile-based-graphical-time-warping
description: Use when XCMS preprocessing leaves suspected misaligned feature groups
  whose RT drift differs across m/z bins or samples, and single per-sample warping
  functions cannot correct them.
when_to_use_negative:
- The input is already a feature table without raw extracted-ion profiles — ncGTW
  realignment operates on profile-level data, not peak tables alone
- 'Features are well aligned: XCMS''s single per-sample warping function already suffices,
  and ncGTW''s false positive rate may be higher than its theoretical 0.05 threshold
  (as reported for the Rotterdam dataset)'
- Peak-shape distortion after realignment is unacceptable and cannot be handled —
  apply ncGTW's post-processing module or consider that parameter tuning may be needed
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: ncGTW
  role: Performs misalignment detection, two-stage reference-free graphical time warping,
    and RT warping-function generation (loadProfile, ncGTWalign, adjustRT)
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Host preprocessing pipeline (peak detection, RT alignment, grouping, peak-filling);
    receives the ncGTW realigned RTs as a plug-in
- name: R
  role: Implementation language; ncGTW is installed as an R/Bioconductor package
  repo: https://github.com/ChiungTingWu/ncGTW
- name: Bioconductor
  role: Distribution channel for the ncGTW R package
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:342270b0bd436fe122faa8e69466ad83a283c9768c775ca0ffcbd7e75531a99d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - Bioconductor
  - R
  - XCMS
  - ncGTW
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# profile-based-graphical-time-warping

## Summary

Realignment of misaligned LC-MS extracted-ion profile features via ncGTW, a reference-free two-stage graphical time warping (combinatorial pairwise warpings aggregated onto a pseudo-reference), used as a plug-in to replace XCMS corrected retention times.

## When to use

Use when XCMS preprocessing leaves suspected misaligned feature groups whose RT drift differs across m/z bins or samples, and single per-sample warping functions cannot correct them.

## When NOT to use

- The input is already a feature table without raw extracted-ion profiles — ncGTW realignment operates on profile-level data, not peak tables alone
- Features are well aligned: XCMS's single per-sample warping function already suffices, and ncGTW's false positive rate may be higher than its theoretical 0.05 threshold (as reported for the Rotterdam dataset)
- Peak-shape distortion after realignment is unacceptable and cannot be handled — apply ncGTW's post-processing module or consider that parameter tuning may be needed

## Inputs

- XCMS object (e.g. xcmsLargeWin) with peak detection, RT alignment and peak grouping already applied
- Raw LC-MS profile files supplying extracted-ion chromatograms for flagged feature groups
- List of misaligned feature groups detected by ncGTW's misalignment detection module

## Outputs

- Sample-specific RT warping functions per realigned feature
- Updated XCMS corrected retention times (xcmsLargeWin@rt$corrected) for realigned features
- Stored realigned feature object (e.g. ncGTW_aligned_features.rda) for downstream XCMS grouping and peak-filling

## How to apply

Load the raw extracted-ion profile data for each XCMS feature group flagged as misaligned using ncGTW's loadProfile on the XCMS object and raw files. Run ncGTWalign on each flagged feature's profiles (e.g. parSamp=5 to split features into sub-groups): Stage 1 solves combinatorial pairwise warping as maximum-flow problems with neighboring-sample structural constraints, and Stage 2 coordinately aggregates all samples onto a virtual pseudo-reference instead of fixing one reference sample. Convert the alignment to sample-specific RT warping functions with adjustRT/rtncGTW and replace xcmsLargeWin@rt$corrected for the realigned features. Store the realigned features and new warping functions (e.g. ncGTW_aligned_features.rda) so XCMS can reuse them for more accurate grouping and peak-filling.

## Related tools

- **ncGTW** (Performs misalignment detection, two-stage reference-free graphical time warping, and RT warping-function generation (loadProfile, ncGTWalign, adjustRT)) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Host preprocessing pipeline (peak detection, RT alignment, grouping, peak-filling); receives the ncGTW realigned RTs as a plug-in)
- **R** (Implementation language; ncGTW is installed as an R/Bioconductor package) — https://github.com/ChiungTingWu/ncGTW
- **Bioconductor** (Distribution channel for the ncGTW R package)

## Examples

```
ncGTWres <- ncGTWalign(ncGTWinput, xcmsLargeWin, parSamp=5); ncGTWwarp <- adjustRT(ncGTWres, xcmsLargeWin, ncGTWparam)
```

## Evaluation signals

- Average pairwise correlation coefficient of realigned profiles is higher than after the initial XCMS alignment on true positive misaligned features
- Average pairwise total overlapping area of realigned profiles is higher than after the initial XCMS alignment
- Post-alignment screening with these two scores removes false positives flagged by the detection module
- Replaced xcmsLargeWin@rt$corrected values feed into XCMS grouping/peak-filling without errors, and previously missing peaks are retrieved by peak-filling
- Realignment succeeds on validated cases: realistic synthetic data and internal QC samples (Rotterdam and MESA iQC)

## Limitations

- ncGTW's misalignment detection module can flag false positives (e.g. a higher-than-theoretical FPR in the Rotterdam dataset), so post-alignment screening of correlation/overlap scores is needed
- Realignment can cause peak shape distortion; correction relies on ncGTW's post-processing module and parameter settings may need adjustment/optimization
- Exactly how structural information of neighboring and distant samples is incorporated is not fully detailed in the paper; exact runtime metrics on large-scale datasets are not provided

## Evidence

- [readme] pairwise alignment with structure constraints: [evidence span withheld — anchor status: external_doc]
- [readme] aggregation of pairwise alignments to warping functions: [evidence span withheld — anchor status: external_doc]
- [other] realignment output replaces XCMS RTs: [evidence span withheld — anchor status: not_found]
