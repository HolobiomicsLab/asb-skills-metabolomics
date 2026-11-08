---
name: r-bioconductor-metabolomics-preprocessing
description: Use when raw LC-MS profile files (e.g. mzXML) must be converted into
  XCMS peak-grouping objects with two bw settings before ncGTW misalignment detection
  can be run.
when_to_use_negative:
- Input is already a feature table or aligned peak matrix rather than raw LC-MS profiles
  — XCMS peak detection and grouping have already been done.
- Only one grouping resolution is needed (e.g. standard XCMS-only analysis without
  ncGTW misalignment detection) — the dual-bw grouping is specific to ncGTW's detection
  criterion.
- The data are not LC-MS metabolomics profiles requiring retention-time alignment
  (e.g. proteomics or non-chromatographic data).
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: XCMS
  role: Performs peak detection (centWave), RT correction (retcor), grouping, and
    peak-filling on raw LC-MS profiles; its two grouping outputs are the required
    inputs for ncGTW
- name: ncGTW
  role: Consumes the two XCMS grouping objects to detect misaligned feature groups
    and realign their profiles, returning realigned data to XCMS
  repo: https://github.com/ChiungTingWu/ncGTW
- name: R
  role: Implementation environment; ncGTW is an R package available via Bioconductor
    or devtools::install_github
  repo: https://github.com/ChiungTingWu/ncGTW
- name: Bioconductor
  role: Distribution channel for installing the ncGTW R package via BiocManager
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:1c4d58c207d5fab7e692b500878ae9bf08718f91b1a8bef2d51cd3c52d424185
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - Bioconductor
  - R
  - XCMS
  - ncGTW
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# r-bioconductor-metabolomics-preprocessing

## Summary

Run the R/Bioconductor XCMS preprocessing pipeline (centWave peak detection, RT correction, dual-resolution grouping) on LC-MS profiles to produce the grouping objects that ncGTW requires for misalignment detection and targeted realignment.

## When to use

Use when raw LC-MS profile files (e.g. mzXML) must be converted into XCMS peak-grouping objects with two bw settings before ncGTW misalignment detection can be run.

## When NOT to use

- Input is already a feature table or aligned peak matrix rather than raw LC-MS profiles — XCMS peak detection and grouping have already been done.
- Only one grouping resolution is needed (e.g. standard XCMS-only analysis without ncGTW misalignment detection) — the dual-bw grouping is specific to ncGTW's detection criterion.
- The data are not LC-MS metabolomics profiles requiring retention-time alignment (e.g. proteomics or non-chromatographic data).

## Inputs

- Raw LC-MS profile files (e.g. 20 mzXML example files from system.file('extdata', package='ncGTW'))
- XCMS parameter settings: centWave ppm=15, peakwidth=c(2,25); grouping bw=6 and bw=0.3, mzwid=0.05

## Outputs

- xcmsLargeWin grouping object (bw=6, mzwid=0.05)
- xcmsSmallWin grouping object (bw=0.3, mzwid=0.05)
- Single output file xcms_grouping_results.RData containing both grouping objects

## How to apply

Load the raw LC-MS profile files into R (for the ncGTW example, 20 mzXML files from system.file('extdata', package='ncGTW')) and detect chromatographic peaks with XCMS centWave using ppm=15 and peakwidth=c(2,25). Correct retention times with the XCMS retcor/alignment module, then group the corrected peaks twice with mzwid=0.05: once with bw=6 (the expected maximal RT drift, yielding xcmsLargeWin) and once with bw=0.3 (near the RT sampling resolution, i.e. the inverse of scan frequency, yielding xcmsSmallWin). Save both grouping objects to a single .RData file, because ncGTW's misalignment detection estimates a p-value per feature from the higher-resolution (bwSmall=0.3) alignment and matches features to the lower-resolution (bw=6) groups. After ncGTW realigns detected misaligned features, feed the realigned data back to XCMS for more accurate grouping and peak-filling.

## Related tools

- **XCMS** (Performs peak detection (centWave), RT correction (retcor), grouping, and peak-filling on raw LC-MS profiles; its two grouping outputs are the required inputs for ncGTW)
- **ncGTW** (Consumes the two XCMS grouping objects to detect misaligned feature groups and realign their profiles, returning realigned data to XCMS) — https://github.com/ChiungTingWu/ncGTW
- **R** (Implementation environment; ncGTW is an R package available via Bioconductor or devtools::install_github) — https://github.com/ChiungTingWu/ncGTW
- **Bioconductor** (Distribution channel for installing the ncGTW R package via BiocManager)

## Examples

```
BiocManager::install("ncGTW"); files <- system.file("extdata", package="ncGTW"); xset <- xcmsSet(files, method="centWave", ppm=15, peakwidth=c(2,25)); xset <- retcor(xset); xcmsLargeWin <- group(xset, bw=6, mzwid=0.05); xcmsSmallWin <- group(xset, bw=0.3, mzwid=0.05); save(xcmsLargeWin, xcmsSmallWin, file="xcms_grouping_results.RData")
```

## Evaluation signals

- The script runs to completion without errors on all 20 example mzXML files.
- Both xcmsLargeWin and xcmsSmallWin contain populated peak-group features with mzwid=0.05.
- The sample-index sets in both grouping objects are consumable by the ncGTW misalignment detection module (features found by high-resolution grouping can be matched to lower-resolution groups).
- xcmsSmallWin used bw=0.3, near the RT sampling resolution (inverse of scan frequency); xcmsLargeWin used bw=6, matching the expected maximal RT drift.
- The output file xcms_grouping_results.RData loads and contains exactly both grouping objects.

## Limitations

- Classic alignment methods like XCMS assume a single time-warping function per sample, so compound-specific and run-order-dependent RT drift may remain uncorrected without ncGTW.
- No specific version numbers for XCMS or ncGTW are provided; ncGTW parameter settings are not exhaustively listed and users must consult the package defaults.
- ncGTW's misalignment detection can flag false positives (higher false positive rate than the theoretical 0.05 threshold in the Rotterdam dataset), requiring later screening via correlation coefficient and overlapping-area scores.

## Evidence

- [other] ncGTW misalignment detection requires two XCMS grouping results with different bw values: [evidence span withheld — anchor status: not_found]
