---
name: extracted-ion-chromatogram-visualization
description: Use when two XCMS grouping results (different bw) have been screened
  by ncGTW misalignDetect and the flagged groups in excluGroups need visual confirmation
  of RT drift via EIC plots.
when_to_use_negative:
- When the goal is to detect or realign misaligned features — use ncGTW misalignDetect/realignment
  instead; the EIC plot only visually verifies detection output and cannot detect
  misalignment on its own.
- When only raw LC-MS profiles without XCMS peak detection and grouping are available
  — the visualization operates on defined XCMS peak groups.
- When no run-order information exists, since the diagnostic signal (contours colored
  by injection order showing progressive apex drift) cannot be interpreted.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3383
tools:
- name: ncGTW
  role: Provides the plotGroup function to draw extracted-ion-chromatogram plots of
    flagged peak groups for visual validation of misalignment detection
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Supplies the two grouping results (different bw) and the peak group definitions
    whose EICs are visualized
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:36f0b7986ae941ff94103f3774f4b49bd982b635b447fcbc147cfb338ef7b401
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - R
  - XCMS
  - ncGTW
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# extracted-ion-chromatogram-visualization

## Summary

Draws extracted-ion-chromatogram (EIC) plots of XCMS peak groups flagged by the ncGTW misalignment detection, coloring peak contours by sample run order, so that compound-specific, run-order-dependent RT drift can be visually confirmed before realignment.

## When to use

Use when two XCMS grouping results (different bw) have been screened by ncGTW misalignDetect and the flagged groups in excluGroups need visual confirmation of RT drift via EIC plots.

## When NOT to use

- When the goal is to detect or realign misaligned features — use ncGTW misalignDetect/realignment instead; the EIC plot only visually verifies detection output and cannot detect misalignment on its own.
- When only raw LC-MS profiles without XCMS peak detection and grouping are available — the visualization operates on defined XCMS peak groups.
- When no run-order information exists, since the diagnostic signal (contours colored by injection order showing progressive apex drift) cannot be interpreted.

## Inputs

- Two XCMS grouping results: xcmsLargeWin (low-resolution grouping, bw = expected maximal RT drift) and xcmsSmallWin (high-resolution grouping, bw near RT sampling resolution)
- excluGroups table of flagged peak groups returned by ncGTW misalignDetect()
- Sample run-order (injection sequence) metadata for contour coloring

## Outputs

- EIC plots per flagged peak group, with peak contours colored by sample run order
- Visual confirmation/denial of run-order-dependent RT drift for each group in excluGroups

## How to apply

After running misalignDetect(xcmsLargeWin, xcmsSmallWin, ppm=15) on two XCMS grouping results — xcmsLargeWin from lower-resolution grouping with the RT window set to the expected maximal RT drift, and xcmsSmallWin from higher-resolution grouping with the RT window near the RT sampling resolution — use the ncGTW plotGroup function to draw EIC plots for each peak group returned in the excluGroups table. Color the peak contours by sample run order: a truly misaligned feature shows its apex RT shifting systematically with injection order, which is the drift pattern that a single sample-level warping function in XCMS cannot correct because RT drift differs across m/z bins. Use the plots to triage the detected groups, since the method intentionally accepts some false positives and screens them out later via post-alignment correlation-coefficient and overlapping-area scores. In the example dataset this visual check should confirm exactly two flagged peak groups, matching the excluGroups.csv written by the detection step.

## Related tools

- **ncGTW** (Provides the plotGroup function to draw extracted-ion-chromatogram plots of flagged peak groups for visual validation of misalignment detection) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Supplies the two grouping results (different bw) and the peak group definitions whose EICs are visualized)

## Evaluation signals

- One EIC plot is produced per row of excluGroups; in the example dataset the plot count equals 2, matching excluGroups.csv.
- In confirmed misaligned groups, peak contours colored by run order show apex RT shifting monotonically with injection order.
- Each flagged group has a disjoint neighbor feature in the same m/z bin, consistent with ncGTW's two detection criteria.
- The plotted groups reproduce the vignette's example figures for the flagged groups in xcmsLargeWin vs xcmsSmallWin.

## Limitations

- Visual inspection is qualitative; the detection decision rests on the order-statistics p-values computed by misalignDetect, not on the plot itself.
- The false positive rate can exceed the theoretical threshold (e.g., in the Rotterdam dataset it is much higher than 0.05), so plotted groups may include false positives that only a post-alignment screening step removes.
- Realigned features can show peak shape distortion, which the ncGTW post-processing module must correct; raw EIC inspection cannot resolve this.

## Evidence

- [intro] An example from the Rotterdam dataset, shows that even with similar RT, the drift of each sample could be significantly different in two m/z bins.: "An example from the Rotterdam dataset, shows that even with similar RT, the drift of each sample could be significantly different in two m/z bins."
- [intro] Classic alignment neglects run order: "classic alignment methods conveniently assume a single warping function across all m/z bins and perform multiple alignment that neglects the run order of each sample"
