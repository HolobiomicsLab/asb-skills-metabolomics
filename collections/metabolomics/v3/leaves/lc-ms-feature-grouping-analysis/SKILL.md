---
name: lc-ms-feature-grouping-analysis
description: Use when LC-MS peak data has been grouped by XCMS at two RT-window (bw)
  resolutions and you need to identify which peak groups (features) are misaligned
  before realignment.
when_to_use_negative:
- Input is a single XCMS grouping result — misalignDetect() requires two grouping
  results produced at different bw (RT window) resolutions to compare
- The goal is to realign already-flagged misaligned features rather than detect them;
  use the ncGTW realignment module instead
- Data has not been preprocessed by XCMS (no peak detection, RT alignment, or peak
  grouping performed) or is not LC-MS metabolomics peak data
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: ncGTW
  role: Provides misalignDetect() (order-statistics p-value test plus disjoint-neighbor
    check in the same m/z bin) and plotGroup() for visual verification of flagged
    peak groups
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Produces the two grouping results (different bw RT windows) whose comparison
    drives misalignment detection; XCMS peak detection, RT alignment, and peak grouping
    precede the skill
- name: R
  role: Execution environment; ncGTW is installed via BiocManager or devtools from
    GitHub
  repo: https://github.com/ChiungTingWu/ncGTW
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:fecdddc101826883a02c2bc341d2a0391fd2d728e23816a15b321352fd11e7ba
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - R
  - XCMS
  - ncGTW
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# lc-ms-feature-grouping-analysis

## Summary

Reproduces LC-MS feature grouping plus ncGTW misalignment detection: run XCMS grouping at two RT-window resolutions and apply ncGTW's misalignDetect() to flag peak groups whose alignment is unreliable, returning the excluGroups table.

## When to use

Use when LC-MS peak data has been grouped by XCMS at two RT-window (bw) resolutions and you need to identify which peak groups (features) are misaligned before realignment.

## When NOT to use

- Input is a single XCMS grouping result — misalignDetect() requires two grouping results produced at different bw (RT window) resolutions to compare
- The goal is to realign already-flagged misaligned features rather than detect them; use the ncGTW realignment module instead
- Data has not been preprocessed by XCMS (no peak detection, RT alignment, or peak grouping performed) or is not LC-MS metabolomics peak data

## Inputs

- Two XCMS grouping result objects from the same LC-MS peak table (xcmsLargeWin: low-resolution grouping with RT window at expected maximal RT drift; xcmsSmallWin: high-resolution grouping with RT window near RT sampling resolution)
- m/z ppm tolerance (e.g. ppm=15 in the vignette example)
- ncGTW package example data or user LC-MS peak data preprocessed by XCMS (peak detection, RT alignment, peak grouping)

## Outputs

- excluGroups table listing peak groups detected as misaligned (exactly two groups for the ncGTW example dataset)
- excluGroups.csv export of the flagged peak groups
- Optional extracted-ion-chromatogram plots from plotGroup colored by sample run order

## How to apply

Produce two XCMS grouping results from the same peak table: xcmsLargeWin with the RT window set to the expected maximal RT drift (low-resolution grouping) and xcmsSmallWin with the RT window near the RT sampling resolution (the inverse of scan frequency; high-resolution grouping). Call ncGTW's misalignDetect(xcmsLargeWin, xcmsSmallWin, ppm=15): it estimates each feature's p-value under a discrete-uniform null by testing whether the feature's sample-index range is too small (order-statistics criterion), and requires disjoint sample subsets among neighboring features in the same m/z bin. Collect the returned excluGroups table of flagged peak groups and write it to CSV. Optionally verify flagged groups visually by drawing extracted-ion-chromatogram plots with plotGroup, coloring peak contours by sample run order.

## Related tools

- **ncGTW** (Provides misalignDetect() (order-statistics p-value test plus disjoint-neighbor check in the same m/z bin) and plotGroup() for visual verification of flagged peak groups) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Produces the two grouping results (different bw RT windows) whose comparison drives misalignment detection; XCMS peak detection, RT alignment, and peak grouping precede the skill)
- **R** (Execution environment; ncGTW is installed via BiocManager or devtools from GitHub) — https://github.com/ChiungTingWu/ncGTW

## Examples

```
excluGroups <- misalignDetect(xcmsLargeWin, xcmsSmallWin, ppm=15); write.csv(excluGroups, "excluGroups.csv")
```

## Evaluation signals

- Saved excluGroups.csv contains exactly two peak groups for the ncGTW example dataset, matching the vignette's reported count
- Each flagged group's sample-index range is too small under the discrete-uniform null (order-statistics p-value criterion)
- Flagged features have disjoint sample subsets among neighboring features in the same m/z bin
- plotGroup EICs of flagged groups show run-order-dependent retention-time drift (peak contours colored by sample run order)
- The two XCMS grouping inputs were genuinely produced at different resolutions: one RT window at expected maximal RT drift, the other near the RT sampling resolution

## Limitations

- False positive rate can exceed the theoretical 0.05 threshold (observed in the Rotterdam dataset); the article suggests screening out false positives in a post-alignment step using correlation coefficient and overlapping area scores
- Peak shape distortion may be introduced by ncGTW processing; parameter settings are not exhaustively listed in the paper and users must consult the package defaults
- Detection quality depends on the contrast between the two grouping resolutions; no version numbers for XCMS or ncGTW are specified in the article

## Evidence
