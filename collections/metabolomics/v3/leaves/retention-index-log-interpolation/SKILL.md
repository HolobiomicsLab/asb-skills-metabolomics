---
name: retention-index-log-interpolation
description: Use when aligned per-sample peaklists (m/z–RT pairs) from multiple LC/HRMS
  samples or batches show systematic RT drift, and no external RT standard ladder
  is available — indigenous recurring peaks serve as the reference markers.
when_to_use_negative:
- Only a single HRMS data file is being processed (no cross-sample/batch drift to
  correct — the article's peak-picking results for MTBLS1684 file 003.d were computed
  without this step being the goal)
- The input is already an aligned, drift-corrected feature table (e.g. from MZmine
  2, xcms-CAMERA or MS-DIAL) — applying equation 5 again would double-correct RTs
- The sample has too few indigenous reference markers bracketing a peak and the median-drift
  fallback is unacceptable for the downstream statistical analysis
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: IDSL.IPA
  role: R package implementing the retention index-based RT correction (dynamic retention
    index marker approach) within its peak alignment workflow
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Runtime environment; the correction is executed as part of the IPA_workflow
    function driven by a parameter spreadsheet
  repo: https://cran.r-project.org/package=IDSL.IPA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:c6e8275f4c9f241892f1b5e0362ff1554b407af68934fcf3a3da3752b6cd9110
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# retention-index-log-interpolation

## Summary

Applies the IDSL.IPA retention index approach (equation 5) to correct systematic retention time drift in untargeted LC/HRMS peaklists by interpolating RT shifts against a dynamic pool of indigenous (endogenous) reference peak markers that bracket each peak in time.

## When to use

Use when aligned per-sample peaklists (m/z–RT pairs) from multiple LC/HRMS samples or batches show systematic RT drift, and no external RT standard ladder is available — indigenous recurring peaks serve as the reference markers.

## When NOT to use

- Only a single HRMS data file is being processed (no cross-sample/batch drift to correct — the article's peak-picking results for MTBLS1684 file 003.d were computed without this step being the goal)
- The input is already an aligned, drift-corrected feature table (e.g. from MZmine 2, xcms-CAMERA or MS-DIAL) — applying equation 5 again would double-correct RTs
- The sample has too few indigenous reference markers bracketing a peak and the median-drift fallback is unacceptable for the downstream statistical analysis

## Inputs

- Per-sample IDSL.IPA peaklists (m/z–RT pairs per HRMS file, e.g. from the MTBLS136, MTBLS1684, ST000923 or ST001237 studies)
- Dynamic pool of indigenous reference peak markers (recurring endogenous m/z–RT pairs clustered across samples)

## Outputs

- Corrected retention time values for every peak in every sample
- A single CSV table holding the corrected RT columns for all samples (the aligned peak table used downstream by IDSL.IPA alignment and IDSL.UFA annotation)

## How to apply

Load the per-sample IDSL.IPA peaklists (m/z–RT pairs per HRMS file, e.g. from an MTBLS1684-style study) and build the reference marker pool by finding recurring m/z–RT pairs across samples that act as retention index markers. For each peak, locate the nearest preceding and following reference markers and use log-interpolation (equation 5) to estimate the local systematic RT drift, then repeat the correction in several steps, re-using the reference peaks to increase statistical significance. If a sample lacks sufficient local markers, fall back to the median drift of the whole reference marker pool. Iterate until corrected RTs converge, then write the corrected retention time column for every sample into a single CSV table. Rationale: the article reports this dynamic-marker index method removed systematic RT drift for the MTBLS136 study (1649 samples) and compares it against polynomial regression methods in the supplementary material.

## Related tools

- **IDSL.IPA** (R package implementing the retention index-based RT correction (dynamic retention index marker approach) within its peak alignment workflow) — https://github.com/idslme/IDSL.IPA
- **R** (Runtime environment; the correction is executed as part of the IPA_workflow function driven by a parameter spreadsheet) — https://cran.r-project.org/package=IDSL.IPA

## Examples

```
IPA_workflow("IPA_parameters.xlsx")
```

## Evaluation signals

- Corrected RTs must reduce systematic RT drift across samples relative to uncorrected RTs, as reported for the MTBLS136 study (n = 1649)
- Corrected RT variation should be compared against (and outperform or match) polynomial-regression RT correction, as plotted in the supplementary figures
- Every peak's corrected RT must be anchored by a preceding and a following reference marker; peaks lacking local markers must fall back to the median pool drift and be flagged
- After correction, the same recurring m/z–RT marker pairs should co-locate in RT across samples, enabling cross-referencing of peaks between samples
- Output must be a single CSV table with one corrected RT column per sample, matching the number of input peaklists

## Limitations

- When a sample lacks sufficient local reference markers, the method falls back to the median drift of the reference marker pool, which is less accurate than local interpolation
- The correction is data-driven from indigenous peaks, so samples with unusual endogenous composition may yield a sparse or biased marker pool
- The pipeline's claims about the correction are demonstrated on specific studies (e.g. MTBLS136, 1649 samples); no explicit limitations of the RT correction itself are discussed in the article

## Evidence
