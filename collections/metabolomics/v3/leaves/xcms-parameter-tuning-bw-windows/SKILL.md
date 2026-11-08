---
name: xcms-parameter-tuning-bw-windows
description: Use when XCMS-preprocessed LC-MS peaks need paired grouping results at
  two RT-window resolutions (large bw = expected maximal RT drift; small bw ≈ inverse
  of scan frequency) as input to ncGTW misalignDetect.
when_to_use_negative:
- When only one XCMS grouping result is available — the two-criterion detection needs
  both a higher-resolution reference grouping and the lower-resolution grouping to
  match neighboring features.
- When the goal is a single standard XCMS feature table for downstream statistics
  rather than misalignment detection/realignment via the ncGTW plug-in.
- When working from raw unprocessed vendor files — peak detection, RT alignment, and
  grouping must first be completed by XCMS before tuning the grouping bw.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: XCMS
  role: Performs peak detection, RT alignment, and produces the two peak-grouping
    results with different bw (RT window) settings
- name: ncGTW
  role: Consumes the paired groupings (xcmsLargeWin, xcmsSmallWin) in misalignDetect
    to flag misaligned feature groups in excluGroups
  repo: https://github.com/ChiungTingWu/ncGTW
- name: Bioconductor
  role: Distribution channel from which the ncGTW R package is installed for this
    workflow
  repo: https://github.com/ChiungTingWu/ncGTW
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:c546eff3a8b093cb73aafc580a98e2a07f8fa2002423164082633a45c4c01819
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
yes_rate: 0.875
n_claims: 8
---

# xcms-parameter-tuning-bw-windows

## Summary

Produces two XCMS peak-grouping results from the same LC-MS peak data by tuning the grouping RT window (bw): a lower-resolution grouping with bw set to the expected maximal RT drift, and a higher-resolution grouping with bw near the RT sampling resolution. The two groupings are the required paired input for ncGTW's misalignment detection.

## When to use

Use when XCMS-preprocessed LC-MS peaks need paired grouping results at two RT-window resolutions (large bw = expected maximal RT drift; small bw ≈ inverse of scan frequency) as input to ncGTW misalignDetect.

## When NOT to use

- When only one XCMS grouping result is available — the two-criterion detection needs both a higher-resolution reference grouping and the lower-resolution grouping to match neighboring features.
- When the goal is a single standard XCMS feature table for downstream statistics rather than misalignment detection/realignment via the ncGTW plug-in.
- When working from raw unprocessed vendor files — peak detection, RT alignment, and grouping must first be completed by XCMS before tuning the grouping bw.

## Inputs

- XCMS-detected peak list from LC-MS centroided data (peak detection and RT alignment already done)
- Scan frequency / RT sampling resolution of the instrument (inverse of scan frequency sets the small-bw window)
- Expected maximal RT drift of the study (sets the large-bw window)
- ppm tolerance for ncGTW matching (ppm=15 in the example)

## Outputs

- xcmsLargeWin: lower-resolution XCMS grouping result (large bw)
- xcmsSmallWin: higher-resolution XCMS grouping result (small bw)
- excluGroups table (e.g. excluGroups.csv) listing peak groups flagged as misaligned by misalignDetect

## How to apply

Run the XCMS preprocessing (peak detection, RT alignment, peak grouping) once, then build two grouping results on the same detected peaks: xcmsLargeWin with the RT window set to the expected maximal retention-time drift, and xcmsSmallWin with the RT window near the RT sampling resolution, i.e. the inverse of the scan frequency. The rationale is that the higher-resolution grouping gives an accurate-alignment reference used by ncGTW to compute each feature's order-statistics p-value under a discrete-uniform null, while the lower-resolution grouping provides the neighboring features matched in the same m/z bin. Only the grouping bandwidth differs between the two objects; peak detection and RT alignment parameters stay fixed so differences are attributable to grouping resolution. The resulting pair is passed to ncGTW's misalignDetect (e.g. ppm=15), which flags features with sufficiently small p-values and disjoint sample subsets, returning them in the excluGroups table.

## Related tools

- **XCMS** (Performs peak detection, RT alignment, and produces the two peak-grouping results with different bw (RT window) settings)
- **ncGTW** (Consumes the paired groupings (xcmsLargeWin, xcmsSmallWin) in misalignDetect to flag misaligned feature groups in excluGroups) — https://github.com/ChiungTingWu/ncGTW
- **Bioconductor** (Distribution channel from which the ncGTW R package is installed for this workflow) — https://github.com/ChiungTingWu/ncGTW

## Examples

```
excluGroups <- misalignDetect(xcmsLargeWin, xcmsSmallWin, ppm=15)
```

## Evaluation signals

- The vignette example dataset must yield exactly two peak groups in excluGroups when misalignDetect(xcmsLargeWin, xcmsSmallWin, ppm=15) is run; saved excluGroups.csv should contain exactly two rows.
- xcmsSmallWin's grouping RT window must be near the RT sampling resolution (the inverse of the scan frequency), not an arbitrary small value.
- xcmsLargeWin and xcmsSmallWin must share identical peak detection and RT alignment results, differing only in grouping bw, so detected misalignment is attributable to grouping resolution.
- Flagged groups must satisfy ncGTW's two criteria: sufficiently small order-statistics p-value under the discrete-uniform null and presence of disjoint sample subsets in the same m/z bin.
- Flagged groups should pass a visual check via plotGroup EIC plots with peak contours colored by sample run order.

## Limitations

- Detection has a nontrivial false positive rate — in the Rotterdam dataset the false positive rate is much higher than the theoretical threshold of 0.05, so flagged features need post-alignment screening via correlation coefficient and overlapping area scores.
- Only features with sufficiently small p-values and disjoint sample subsets are detected; misaligned features failing these criteria are missed.
- ncGTW is a plug-in, not a standalone tool — it presumes an XCMS grouping output and depends on XCMS for downstream grouping and peak-filling with the realigned data.
- Peak shape distortion can occur after realignment and requires ncGTW's post-processing module to correct; parameter settings are not exhaustively documented and defaults must be consulted in the package.

## Evidence
