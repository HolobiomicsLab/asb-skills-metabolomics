---
name: maximum-flow-warping-estimation
description: Use when XCMS-corrected retention times remain misaligned for specific
  detected feature groups (e.g., flagged by ncGTW misalignment detection) and compound-specific,
  per-feature warping functions are needed instead of one global per-sample warp.
when_to_use_negative:
- The input is already an aligned feature table with no raw extracted-ion profiles
  available — ncGTW warping estimation operates on profile-level data, not peak tables.
- All features are well aligned by the standard XCMS single per-sample warping function
  and misalignment detection flagged no groups, so there is nothing to realign.
- A single global time-warping function per sample is sufficient (no compound-specific,
  m/z-bin-dependent RT drift to correct).
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ncGTW
  role: Provides loadProfile, ncGTWalign (two-stage max-flow pairwise warping plus
    pseudo-reference aggregation), and adjustRT/rtncGTW to produce and apply the RT
    warping functions
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Supplies the misaligned feature groups, the corrected RT slot (xcmsLargeWin@rt$corrected)
    that warping functions replace, and consumes realigned data for grouping and peak-filling
- name: R
  role: Implementation language of the ncGTW plug-in distributed via Bioconductor/GitHub
  repo: https://github.com/ChiungTingWu/ncGTW
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:dbed383ac6ed25ac0cc56670753e6276cf6d72e6cf961a41c48a3f44fad09e87
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - R
  - XCMS
  - ncGTW
  edam_operation: http://edamontology.org/operation_3204
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

# maximum-flow-warping-estimation

## Summary

Reference-free LC-MS retention-time warping estimation from ncGTW: solve combinatorial pairwise profile warpings via maximum-flow problems with neighboring-sample structural constraints, then aggregate all samples onto a virtual pseudo-reference to yield per-feature, sample-specific RT warping functions.

## When to use

Use when XCMS-corrected retention times remain misaligned for specific detected feature groups (e.g., flagged by ncGTW misalignment detection) and compound-specific, per-feature warping functions are needed instead of one global per-sample warp.

## When NOT to use

- The input is already an aligned feature table with no raw extracted-ion profiles available — ncGTW warping estimation operates on profile-level data, not peak tables.
- All features are well aligned by the standard XCMS single per-sample warping function and misalignment detection flagged no groups, so there is nothing to realign.
- A single global time-warping function per sample is sufficient (no compound-specific, m/z-bin-dependent RT drift to correct).

## Inputs

- XCMS object with detected features and corrected retention times, plus raw LC-MS profile files (extracted-ion profiles)
- List of feature groups flagged as misaligned by the ncGTW misalignment detection module
- parSamp sub-grouping parameter (parSamp=5 in the reference workflow)

## Outputs

- Per-feature, sample-specific RT warping functions replacing xcmsLargeWin@rt$corrected
- Realigned feature profiles stored (e.g., ncGTW_aligned_features.rda) for downstream XCMS grouping and peak-filling
- Post-alignment validation scores: average pairwise correlation coefficient and average pairwise total overlapping area

## How to apply

First load the raw extracted-ion LC-MS profiles for each misaligned feature group flagged by ncGTW's detection step, using loadProfile on the XCMS object and the raw files. Run ncGTWalign on each flagged feature's profiles with parSamp=5 (splitting features into sub-groups): Stage 1 performs combinatorial pairwise alignments between every two samples, solving maximum-flow problems that incorporate structural information from neighboring and distant samples with different RT drift patterns; Stage 2 coordinately aggregates all pairwise alignments as constraints to estimate warping functions of all samples to a common coordinate (a virtual reference), avoiding a fixed reference sample. Then apply adjustRT/rtncGTW to convert the two-stage alignment into sample-specific RT warping functions that replace xcmsLargeWin@rt$corrected. Store the realigned features and their new warping functions (e.g., in ncGTW_aligned_features.rda) so XCMS can reuse them for re-grouping and peak-filling. Validate by comparing average pairwise correlation coefficient and average pairwise total overlapping area against the original XCMS alignment; ncGTW should score higher on true-positive misaligned features.

## Related tools

- **ncGTW** (Provides loadProfile, ncGTWalign (two-stage max-flow pairwise warping plus pseudo-reference aggregation), and adjustRT/rtncGTW to produce and apply the RT warping functions) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Supplies the misaligned feature groups, the corrected RT slot (xcmsLargeWin@rt$corrected) that warping functions replace, and consumes realigned data for grouping and peak-filling)
- **R** (Implementation language of the ncGTW plug-in distributed via Bioconductor/GitHub) — https://github.com/ChiungTingWu/ncGTW

## Examples

```
library(ncGTW); prof <- loadProfile(xcmsLargeWin, filePaths); aln <- ncGTWalign(prof, parSamp=5); rt <- adjustRT(xcmsLargeWin, aln); save(rt, file='ncGTW_aligned_features.rda')
```

## Evaluation signals

- Average pairwise correlation coefficient of the realigned feature profiles is higher than for the initial XCMS alignment on true-positive misaligned features (as reported for Rotterdam and MESA iQC samples).
- Average pairwise total overlapping area of the realigned profiles increases relative to the pre-realignment XCMS result.
- adjustRT output actually replaces xcmsLargeWin@rt$corrected — corrected RTs of the targeted features change while other features remain untouched.
- Realigned features and their warping functions are persisted (ncGTW_aligned_features.rda) and can be fed back to XCMS for grouping and peak-filling without errors.
- Peak-shape distortion is checked post-realignment; distorted peaks are handled by ncGTW's post-processing module before downstream use.

## Limitations

- Realignment can introduce peak distortion; ncGTW includes a post-processing module to correct it, and parameter settings can be adjusted to reduce the likelihood of distortion, but defaults may not be optimal.
- Misalignment detection can produce false positives (false positive rate in the Rotterdam dataset was higher than the theoretical 0.05 threshold); a post-alignment screening step using correlation coefficient and overlapping area scores is needed to filter true positives.
- Runtime/computational cost on very large datasets is not characterized in the paper.
- Applicable only as a targeted plug-in for misaligned groups detected from XCMS output — not a standalone end-to-end preprocessing pipeline.

## Evidence

- [readme] Pairwise alignments as constraints to a common coordinate: [evidence span withheld — anchor status: external_doc]
