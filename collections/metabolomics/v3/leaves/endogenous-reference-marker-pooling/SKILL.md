---
name: endogenous-reference-marker-pooling
description: Use when processing untargeted LC/HRMS peaklists from multiple samples
  or batches where systematic retention time drift must be corrected without spiked
  standards, using endogenous peaks as retention index markers.
when_to_use_negative:
- Input is already an aligned, RT-corrected feature table from another pipeline (e.g.
  MZmine 2, xcms-CAMERA, MS-DIAL output)
- Analysis of a single HRMS file with no cross-sample recurrence — the marker pool
  relies on recurring m/z–RT pairs across many samples, so no reliable reference markers
  can be built
- Data lacking endogenous markers across the chromatographic range, where only the
  median-drift fallback would apply and per-peak interpolation cannot be performed
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.IPA
  role: R package implementing the dynamic retention index marker approach for RT
    correction and cross-referencing of peaks across samples
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Runtime environment; IDSL.IPA is distributed via CRAN and invoked through
    the IPA_workflow function
  repo: https://cran.r-project.org/package=IDSL.IPA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:05f8c96ed846db1b12b69a787ddc8ac2cdb7a12389a62e89e6d43d6c4557ab47
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# endogenous-reference-marker-pooling

## Summary

Builds a dynamic pool of indigenous (endogenous) reference peak markers from recurring m/z–RT pairs across IDSL.IPA per-sample peaklists and uses them to correct systematic retention time drift across samples and batches, instead of external standards.

## When to use

Use when processing untargeted LC/HRMS peaklists from multiple samples or batches where systematic retention time drift must be corrected without spiked standards, using endogenous peaks as retention index markers.

## When NOT to use

- Input is already an aligned, RT-corrected feature table from another pipeline (e.g. MZmine 2, xcms-CAMERA, MS-DIAL output)
- Analysis of a single HRMS file with no cross-sample recurrence — the marker pool relies on recurring m/z–RT pairs across many samples, so no reliable reference markers can be built
- Data lacking endogenous markers across the chromatographic range, where only the median-drift fallback would apply and per-peak interpolation cannot be performed

## Inputs

- Per-sample IDSL.IPA peaklists (m/z–RT pairs per LC/HRMS file)
- Aligned peak table across samples/batches (multi-batch study, e.g. n > 100 samples)
- IPA parameter spreadsheet consumed by the IPA_workflow function

## Outputs

- Corrected retention time columns for every sample in a single CSV table
- Cross-referenced aligned peak matrix with drift-corrected RTs suitable for downstream annotation (e.g. IDSL.UFA)

## How to apply

Load the per-sample IDSL.IPA peaklists (m/z–RT pairs extracted from each HRMS file) and cluster recurring m/z–RT pairs across samples to assemble a dynamic pool of endogenous reference markers. For each peak, locate the nearest preceding and following reference markers and apply the log-interpolation of equation (5) to estimate the systematic retention time drift; repeat the correction in several steps with the reference peaks to increase statistical significance. If a sample lacks sufficient local markers, fall back to the median drift of the reference marker pool. This dynamic retention index marker approach is a core IDSL.IPA step (implemented alongside EIC candidate generation, peak detection, recursive mass correction, and annotation in the R package) and is validated by checking that corrected RTs show less systematic drift than uncorrected RTs, and comparing variation against polynomial regression-based RT correction on the same data.

## Related tools

- **IDSL.IPA** (R package implementing the dynamic retention index marker approach for RT correction and cross-referencing of peaks across samples) — https://github.com/idslme/IDSL.IPA
- **R** (Runtime environment; IDSL.IPA is distributed via CRAN and invoked through the IPA_workflow function) — https://cran.r-project.org/package=IDSL.IPA

## Examples

```
library(IDSL.IPA); IPA_workflow("IPA_parameters.xlsx")
```

## Evaluation signals

- Corrected retention times must show reduced systematic RT drift across samples compared with uncorrected RTs (as demonstrated for MTBLS136, n=1649)
- Every peak should be anchored between a nearest preceding and a following reference marker; peaks without local markers must fall back to the median drift of the marker pool
- Corrected RTs should be output for all samples in a single aligned CSV table with consistent column structure
- Compare the variation in corrected RTs against polynomial regression RT correction on the same study as a sanity check

## Limitations

- The approach requires a sufficient density of recurring indigenous reference peaks per sample; samples lacking local markers rely on the median-drift fallback and lose per-peak precision
- Reported validation covers population-scale untargeted LC/HRMS studies (200–1,600 samples; MTBLS136 with 1649 samples); behavior outside this regime is not characterized in the article
- The article does not report quantitative performance metrics for the RT correction itself beyond drift removal and comparison with polynomial regression

## Evidence
