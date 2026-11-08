---
name: lc-ms-retention-time-alignment
description: Use when XCMS-class single-warping-function alignment has produced misaligned
  feature groups in LC-MS profiles — detected by ncGTW's p-value-based misalignment
  detection — and the goal is compound-specific realignment before re-grouping and
  peak-filling.
when_to_use_negative:
- Features are well-aligned by XCMS's single per-sample warping function — ncGTW targets
  only detected misaligned groups, and over-warping risks peak shape distortion (the
  paper notes distortion must be corrected by a post-processing module).
- Input is already a feature/peak table without raw profile data — ncGTW realignment
  operates on extracted-ion profiles (loadProfile), so profile-level raw data must
  be available.
- A fixed-reference alignment is required instead of a reference-free one — ncGTW
  deliberately avoids setting a certain reference.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ncGTW
  role: Detects misaligned feature groups from XCMS output and realigns their profiles
    via two-stage reference-free graphical time warping; provides loadProfile, ncGTWalign,
    adjustRT, and a post-processing module for peak distortion correction
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: 'Upstream/downstream engine: performs peak detection, RT alignment, peak grouping,
    and then uses ncGTW''s realigned data for more accurate grouping and peak-filling'
- name: R
  role: Implementation environment; ncGTW is an R package installed via devtools/BiocManager
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
  source_content_hash: sha256:c0125cb2e740b50de5205191c4d03a84e5e9fe252ed4466efeec20861d4b29ad
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
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# lc-ms-retention-time-alignment

## Summary

Targeted reference-free retention-time realignment of misaligned LC-MS features using ncGTW as a plug-in to XCMS: two-stage combinatorial pairwise profile warping aggregated to a pseudo-reference, yielding sample-specific RT warping functions that replace XCMS corrected RTs.

## When to use

Use when XCMS-class single-warping-function alignment has produced misaligned feature groups in LC-MS profiles — detected by ncGTW's p-value-based misalignment detection — and the goal is compound-specific realignment before re-grouping and peak-filling.

## When NOT to use

- Features are well-aligned by XCMS's single per-sample warping function — ncGTW targets only detected misaligned groups, and over-warping risks peak shape distortion (the paper notes distortion must be corrected by a post-processing module).
- Input is already a feature/peak table without raw profile data — ncGTW realignment operates on extracted-ion profiles (loadProfile), so profile-level raw data must be available.
- A fixed-reference alignment is required instead of a reference-free one — ncGTW deliberately avoids setting a certain reference.

## Inputs

- XCMS feature/peak group objects (e.g., xcmsLargeWin with peak detection, RT correction, and grouping done) plus the raw LC-MS profile files
- Extracted-ion chromatogram profiles for samples in each misaligned feature group (loaded via ncGTW loadProfile)
- Misaligned feature groups flagged by ncGTW's misalignment detection module

## Outputs

- Sample-specific RT warping functions per realigned feature (from adjustRT) replacing XCMS corrected retention times
- Realigned feature profiles stored for pass-back to XCMS (e.g., ncGTW_aligned_features.rda) for re-grouping and peak-filling
- Post-alignment quality scores (average pairwise correlation coefficient and average pairwise total overlapping area) for true/false-positive screening

## How to apply

Run XCMS peak detection, RT correction, and grouping first, then let ncGTW's misalignment detection module flag feature groups (p-value estimated from higher-resolution alignment; features with sufficiently small p-values and disjoint sample subsets are considered). For each flagged group, load the raw extracted-ion profiles of all samples with loadProfile, then run ncGTWalign with parSamp=5 (features split into sub-groups): Stage 1 solves all pairwise graphical-time-warping problems between samples with neighboring-sample structural constraints as maximum-flow problems; Stage 2 aggregates all pairwise constraints to estimate each sample's warping function to a common coordinate (a pseudo-reference, no fixed reference sample). Convert the alignment into per-sample warping functions with adjustRT and overwrite the XCMS corrected RTs (xcmsLargeWin@rt$corrected), then hand the realigned features back to XCMS for more accurate grouping and peak-filling so missing peaks can be retrieved. Judge success by higher average pairwise correlation coefficient and average pairwise total overlapping area than the initial XCMS alignment on true positive misaligned features (as reported on Rotterdam and MESA iQC samples), and screen false positives via a post-alignment step using those two scores.

## Related tools

- **ncGTW** (Detects misaligned feature groups from XCMS output and realigns their profiles via two-stage reference-free graphical time warping; provides loadProfile, ncGTWalign, adjustRT, and a post-processing module for peak distortion correction) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Upstream/downstream engine: performs peak detection, RT alignment, peak grouping, and then uses ncGTW's realigned data for more accurate grouping and peak-filling)
- **R** (Implementation environment; ncGTW is an R package installed via devtools/BiocManager) — https://github.com/ChiungTingWu/ncGTW
- **Bioconductor** (Distribution channel for the ncGTW R package)

## Examples

```
devtools::install_github("ChiungTingWu/ncGTW"); aligned <- ncGTWalign(loadProfile(xcmsLargeWin, misFeat), xcmsLargeWin, parSamp=5); xcmsLargeWin@rt$corrected <- adjustRT(aligned)
```

## Evaluation signals

- Average pairwise correlation coefficient of the realigned feature profiles is higher than after the initial XCMS alignment on true positive misaligned features.
- Average pairwise total overlapping area of aligned peaks improves over XCMS's single-warping-function result.
- Post-alignment screening via the two performance scores correctly separates true positives from false positives among detected misaligned features.
- XCMS peak-filling after realignment retrieves missing peaks that were absent under the original alignment.
- New warping functions are per-sample and compound-specific (differ across m/z bins), not a single global warping function per sample.

## Limitations

- A higher theoretical false positive rate is accepted by the detection module and screened out later; the paper notes a relatively higher false positive rate in the Rotterdam dataset than the theoretical threshold of 0.05.
- Realignment can induce peak shape distortion; it must be corrected by ncGTW's post-processing module, and parameter settings can be adjusted to reduce distortion likelihood (defaults are not exhaustively documented).
- Single per-sample warping methods cannot correct features whose true warping functions differ over different m/z bins — but if a feature's misalignment is not detectable (p-value not small), it will never be realigned.
- No specific XCMS/ncGTW version numbers, exhaustive parameter lists, or exact runtime metrics are provided in the paper.

## Evidence

- [readme] Pairwise alignments with structural constraints as the core of realignment: [evidence span withheld — anchor status: external_doc]
