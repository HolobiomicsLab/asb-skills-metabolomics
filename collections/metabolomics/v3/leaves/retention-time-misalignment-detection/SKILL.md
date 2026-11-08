---
name: retention-time-misalignment-detection
description: Use when you have two XCMS grouping results (one with an RT window near
  the RT sampling resolution, one with the expected maximal RT drift) and need to
  identify which peak groups are misaligned before realignment.
when_to_use_negative:
- You only have a single XCMS grouping result — detection requires the comparison
  between a higher-resolution and a lower-resolution grouping.
- Input is raw LC-MS spectra that have not undergone XCMS peak detection and peak
  grouping; ncGTW operates on XCMS grouping output as a plug-in.
- Alignment quality is already verified and no misaligned features are suspected;
  detection would only add the false positives that the article notes must be screened
  out later.
tools:
- name: ncGTW
  role: Provides the misalignDetect() function that tests each XCMS feature for misalignment
    via order-statistics p-values and disjoint neighbor checks, returning the excluGroups
    table.
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Produces the peak grouping results (higher- and lower-resolution RT windows)
    that serve as the input objects for misalignDetect().
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:d57eb9524e1deb4224af076dd5130f617265cbaae28b61d3a7eef1296fdba2a4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
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

# retention-time-misalignment-detection

## Summary

Detects misaligned feature groups in LC-MS data using the ncGTW misalignment detection module, which flags XCMS peak groups whose RT alignment is unreliable by comparing a higher-resolution grouping against a lower-resolution one with order-statistics p-values. This is the first stage of the ncGTW plug-in workflow before targeted realignment.

## When to use

Use when you have two XCMS grouping results (one with an RT window near the RT sampling resolution, one with the expected maximal RT drift) and need to identify which peak groups are misaligned before realignment.

## When NOT to use

- You only have a single XCMS grouping result — detection requires the comparison between a higher-resolution and a lower-resolution grouping.
- Input is raw LC-MS spectra that have not undergone XCMS peak detection and peak grouping; ncGTW operates on XCMS grouping output as a plug-in.
- Alignment quality is already verified and no misaligned features are suspected; detection would only add the false positives that the article notes must be screened out later.

## Inputs

- Higher-resolution XCMS grouping result (xcmsSmallWin; RT window near the RT sampling resolution)
- Lower-resolution XCMS grouping result (xcmsLargeWin; RT window set to expected maximal RT drift)
- ppm tolerance parameter (ppm=15 in the example)

## Outputs

- excluGroups table: peak groups (features) flagged as misaligned, e.g. saved as excluGroups.csv
- Optional plotGroup EIC plots with peak contours colored by sample run order for visual verification

## How to apply

Load the ncGTW example or user data containing two XCMS grouping objects: a higher-resolution grouping whose RT window is near the RT sampling resolution (inverse of scan frequency) and a lower-resolution grouping whose RT window is set to the expected maximal RT drift. Run misalignDetect(xcmsLargeWin, xcmsSmallWin, ppm=15), which estimates each feature's p-value under a discrete-uniform null via order statistics on the sample-index range, and additionally checks for neighbor features that are disjoint in the same m/z bin. Features with sufficiently small p-values and disjoint sample subsets are matched to corresponding features from the lower-resolution grouping and returned as the excluGroups table of misaligned peak groups. In the vignette example this table contains exactly two peak groups; write it to CSV (excluGroups.csv) and optionally inspect the flagged groups by drawing extracted-ion-chromatogram plots with plotGroup, coloring peak contours by sample run order.

## Related tools

- **ncGTW** (Provides the misalignDetect() function that tests each XCMS feature for misalignment via order-statistics p-values and disjoint neighbor checks, returning the excluGroups table.) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Produces the peak grouping results (higher- and lower-resolution RT windows) that serve as the input objects for misalignDetect().)

## Examples

```
excluGroups <- misalignDetect(xcmsLargeWin, xcmsSmallWin, ppm = 15); write.csv(excluGroups, "excluGroups.csv")
```

## Evaluation signals

- excluGroups.csv contains exactly two peak groups when applied to the package example with ppm=15, matching the vignette-reported count.
- Flagged features have sufficiently small p-values under the discrete-uniform null hypothesis with accurate alignment (order statistics on sample-index range).
- Flagged groups satisfy the disjoint-sample-subsets / disjoint neighbor feature criterion in the same m/z bin.
- Visual check: plotGroup EIC plots of flagged groups show inconsistent peak RT ordering across sample run order.
- In the paper's large-scale runs, the misalignment detection module flagged a known count (e.g. 57 features detected as potentially misaligned), so counts should be reproducible on the same inputs.

## Limitations

- The false positive rate can exceed the theoretical threshold (e.g. in the Rotterdam dataset it is much higher than 0.05); flagged features may need post-alignment screening via correlation coefficient and overlapping area scores.
- Detection is a filter step that cannot itself correct misalignment, and ncGTW realignment can introduce peak shape distortion, which must be handled by a separate post-processing module.
- Parameter settings are not exhaustively documented; users must consult the package defaults, and no specific XCMS or ncGTW versions are stated in the article.

## Evidence
