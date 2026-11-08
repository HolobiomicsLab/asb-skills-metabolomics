---
name: xcms-peak-grouping-parameterization
description: 'Use when preparing XCMS preprocessing outputs for ncGTW misalignment
  detection: after centWave peak detection (ppm=15, peakwidth=c(2,25)) and retcor,
  run group() twice (bw=6 and bw=0.3, mzwid=0.05) on the same sample set.'
when_to_use_negative:
- The input is already an ncGTW-realigned feature table or a final feature matrix
  — no re-grouping of raw profiles is needed
- Only a single standard XCMS grouping is required (no misalignment detection planned)
  — a two-bw grouping strategy is unnecessary overhead
- Raw chromatographic profiles are unavailable and only peak tables exist — the grouping
  must be run on retcor-corrected peaks within XCMS, not reconstructed post hoc
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: XCMS
  role: Performs the peak detection (centWave), retcor RT correction, and the two
    group() runs (bw=6 and bw=0.3, mzwid=0.05) that supply the grouping results consumed
    by ncGTW
- name: ncGTW
  role: Downstream plug-in that consumes the two grouping results (xcmsLargeWin and
    xcmsSmallWin) to detect misaligned feature groups and realign them
  repo: https://github.com/ChiungTingWu/ncGTW
- name: R
  role: Runtime environment (Bioconductor) in which XCMS grouping objects are created
    and saved to xcms_grouping_results.RData
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:d60817fa8d7618eb6019b16e864098698bbdb9de0b2a58e77e25fda4b387b594
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# xcms-peak-grouping-parameterization

## Summary

Produces the two XCMS peak-grouping results (a large-bw and a small-bw grouping) that ncGTW requires to detect misaligned features in LC-MS data: grouping with bw equal to the expected maximal RT drift and with bw near the RT sampling resolution, both with mzwid=0.05.

## When to use

Use when preparing XCMS preprocessing outputs for ncGTW misalignment detection: after centWave peak detection (ppm=15, peakwidth=c(2,25)) and retcor, run group() twice (bw=6 and bw=0.3, mzwid=0.05) on the same sample set.

## When NOT to use

- The input is already an ncGTW-realigned feature table or a final feature matrix — no re-grouping of raw profiles is needed
- Only a single standard XCMS grouping is required (no misalignment detection planned) — a two-bw grouping strategy is unnecessary overhead
- Raw chromatographic profiles are unavailable and only peak tables exist — the grouping must be run on retcor-corrected peaks within XCMS, not reconstructed post hoc

## Inputs

- Raw LC-MS profile files (e.g. 20 ncGTW example mzXML files from system.file("extdata", package="ncGTW")) sorted by acquisition order
- XCMS centWave peak-detection result (ppm=15, peakwidth=c(2,25))
- XCMS retcor RT-corrected peak table / grouped peak set

## Outputs

- xcmsLargeWin: XCMS grouping object from group(bw=6, mzwid=0.05)
- xcmsSmallWin: XCMS grouping object from group(bw=0.3, mzwid=0.05)
- xcms_grouping_results.RData containing both grouping objects

## How to apply

After centWave peak detection (ppm=15, peakwidth=c(2,25)) and retcor RT correction in XCMS on raw LC-MS profiles (e.g. the 20 ncGTW example mzXML files loaded in acquisition order), run XCMS group() twice on the corrected peaks. First grouping uses the lower-resolution RT window bw=6, chosen to the expected maximal RT drift, yielding the xcmsLargeWin object; second grouping uses the higher-resolution RT window bw=0.3, chosen near the RT sampling resolution (the inverse of scan frequency), yielding the xcmsSmallWin object. Both runs use the m/z bin width mzwid=0.05. ncGTW needs these two groupings because its misalignment detection compares the higher-resolution alignment result against the lower-resolution one: it estimates a p-value per feature under accurate alignment, and matches neighboring features to the corresponding features produced by the lower-resolution grouping. Save both grouping objects (e.g. xcmsLargeWin and xcmsSmallWin) into a single RData file such as xcms_grouping_results.RData for downstream ncGTW consumption.

## Related tools

- **XCMS** (Performs the peak detection (centWave), retcor RT correction, and the two group() runs (bw=6 and bw=0.3, mzwid=0.05) that supply the grouping results consumed by ncGTW)
- **ncGTW** (Downstream plug-in that consumes the two grouping results (xcmsLargeWin and xcmsSmallWin) to detect misaligned feature groups and realign them) — https://github.com/ChiungTingWu/ncGTW
- **R** (Runtime environment (Bioconductor) in which XCMS grouping objects are created and saved to xcms_grouping_results.RData)

## Examples

```
xcmsLargeWin <- group(xset, bw=6, mzwid=0.05); xcmsSmallWin <- group(xset, bw=0.3, mzwid=0.05); save(xcmsLargeWin, xcmsSmallWin, file='xcms_grouping_results.RData')
```

## Evaluation signals

- Both xcmsLargeWin and xcmsSmallWin objects exist and contain populated peak-group features after group() completes
- The sample-index sets in the grouping objects are consumable by the ncGTW misalignment detection module (features with disjoint sample subsets identifiable from the small-bw grouping)
- bw=0.3 is close to the RT sampling resolution (inverse of scan frequency); if scans are coarser/finer than assumed, verify this value against the instrument
- mzwid=0.05 is identical in both grouping runs so that feature matching between resolutions is consistent
- The script runs to completion and both grouping objects are saved together in xcms_grouping_results.RData

## Limitations

- The bw=6 and bw=0.3 values are dataset-specific choices (expected maximal RT drift and RT sampling resolution); other instruments or datasets may require retuning these windows
- No specific XCMS or ncGTW versions were provided; the retcor/group() interface reflects the classic XCMS API and may need translation for newer XCMS3 classes
- Grouping parameters (bw, mzwid) are not exhaustively optimized in the paper; users must consult the package defaults and adjust for their data

## Evidence
