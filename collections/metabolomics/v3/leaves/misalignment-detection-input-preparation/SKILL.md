---
name: misalignment-detection-input-preparation
description: Use when raw LC-MS profiles (e.g. mzXML) must be preprocessed by XCMS
  into two grouping objects — one at large bw, one near RT sampling resolution — so
  ncGTW can detect misaligned features.
when_to_use_negative:
- Input is already a peak/feature table with no raw chromatographic profiles, so centWave/retcor
  cannot be run and the profile-level grouping objects cannot be formed
- The downstream plan does not use ncGTW (e.g. another alignment tool that does not
  require paired low/high-resolution grouping results)
- Only one grouping granularity is needed because no misalignment detection is planned
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: XCMS
  role: 'Performs the required preprocessing: centWave peak detection (ppm=15, peakwidth=c(2,25)),
    retcor RT correction, and group() runs at bw=6 and bw=0.3 with mzwid=0.05'
- name: ncGTW
  role: Consumes the two XCMS grouping results as inputs to its misalignment detection
    module
  repo: https://github.com/ChiungTingWu/ncGTW
- name: R
  role: Execution environment; ncGTW is an R package plug-in for XCMS available via
    Bioconductor or devtools
  repo: https://github.com/ChiungTingWu/ncGTW
- name: Bioconductor
  role: Distribution channel for installing the ncGTW R package (BiocManager::install("ncGTW"))
  repo: https://github.com/ChiungTingWu/ncGTW
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:4ef6ad4599191ff15283ed07544fd976f1f295d84973338583335c6d127850e1
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
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# misalignment-detection-input-preparation

## Summary

Prepares the XCMS preprocessing outputs (centWave peak detection, retcor RT correction, and two group() results at different bw values) that the ncGTW misalignment-detection module consumes before it can flag and realign misaligned LC-MS feature groups.

## When to use

Use when raw LC-MS profiles (e.g. mzXML) must be preprocessed by XCMS into two grouping objects — one at large bw, one near RT sampling resolution — so ncGTW can detect misaligned features.

## When NOT to use

- Input is already a peak/feature table with no raw chromatographic profiles, so centWave/retcor cannot be run and the profile-level grouping objects cannot be formed
- The downstream plan does not use ncGTW (e.g. another alignment tool that does not require paired low/high-resolution grouping results)
- Only one grouping granularity is needed because no misalignment detection is planned

## Inputs

- Raw LC-MS profile files (mzXML/mzML), e.g. 20 example files from system.file("extdata", package="ncGTW")
- Acquisition order of the samples (for ordering before processing)

## Outputs

- xcmsLargeWin: XCMS grouping object with bw=6, mzwid=0.05 (expected maximal RT drift scale)
- xcmsSmallWin: XCMS grouping object with bw=0.3, mzwid=0.05 (near RT sampling resolution)
- Single RData file (xcms_grouping_results.RData) containing both grouping objects

## How to apply

Load the raw LC-MS profile files (e.g. the 20 example mzXML files from system.file("extdata", package="ncGTW")) sorted by acquisition order and detect chromatographic peaks with XCMS centWave using ppm=15 and peakwidth=c(2,25). Correct retention times with XCMS retcor, then run group() twice with mzwid=0.05: once with the larger bw=6 (matching the expected maximal RT drift) to produce xcmsLargeWin, and once with bw=0.3 (near the RT sampling resolution, the inverse of scan frequency) to produce xcmsSmallWin. ncGTW's detection uses the higher-resolution grouping to estimate per-feature p-values under the null of accurate alignment and the lower-resolution grouping to match neighboring features, so both objects must be generated on the same corrected peak set. Save both grouping objects together (e.g. to xcms_grouping_results.RData) so the detection module can access their sample-index sets.

## Related tools

- **XCMS** (Performs the required preprocessing: centWave peak detection (ppm=15, peakwidth=c(2,25)), retcor RT correction, and group() runs at bw=6 and bw=0.3 with mzwid=0.05)
- **ncGTW** (Consumes the two XCMS grouping results as inputs to its misalignment detection module) — https://github.com/ChiungTingWu/ncGTW
- **R** (Execution environment; ncGTW is an R package plug-in for XCMS available via Bioconductor or devtools) — https://github.com/ChiungTingWu/ncGTW
- **Bioconductor** (Distribution channel for installing the ncGTW R package (BiocManager::install("ncGTW"))) — https://github.com/ChiungTingWu/ncGTW

## Examples

```
xset <- xcmsSet(fls, method="centWave", ppm=15, peakwidth=c(2,25)); xset <- retcor(xset); xcmsLargeWin <- group(xset, bw=6, mzwid=0.05); xcmsSmallWin <- group(xset, bw=0.3, mzwid=0.05)
```

## Evaluation signals

- Both xcmsLargeWin and xcmsSmallWin objects exist and contain populated peak-group features with sample-index sets consumable by ncGTW detection
- Both group() runs used the same mzwid=0.05 and the same retcor-corrected peak set, differing only in bw (6 vs 0.3)
- centWave parameters traceable as ppm=15 and peakwidth=c(2,25) on files ordered by acquisition order
- Output file xcms_grouping_results.RData loads in R and exposes both named objects
- bwSmall=0.3 is at or below the RT sampling resolution (inverse of scan frequency) for the instrument's scan rate

## Limitations

- bw=6 must be set to the expected maximal RT drift, which is dataset-dependent and must be chosen per experiment; bwSmall should track the instrument's RT sampling resolution
- No specific version numbers for XCMS or ncGTW are given; ncGTW parameter settings are not exhaustively documented and users must consult the package for defaults
- Misalignment detection has a non-trivial false positive rate (in the Rotterdam dataset it exceeded the theoretical 0.05 threshold), so detection results from downstream ncGTW may require post-realignment screening

## Evidence
