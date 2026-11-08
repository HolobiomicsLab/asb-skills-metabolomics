---
name: lc-hrms-batch-correction-analysis
description: Use when aligning per-sample IDSL.IPA peaklists (m/z–RT pairs from LC/HRMS
  files) across a population-scale study where systematic retention time drift between
  batches or samples must be removed before peak cross-referencing.
when_to_use_negative:
- Input is already an aligned, retention-time-corrected feature table — re-applying
  the index correction would re-drift the aligned RTs
- Samples lack recurring endogenous reference peaks (few overlapping m/z–RT markers),
  forcing reliance on the median-drift fallback with weak correction power
- The goal is chromatographic peak detection or quality filtering rather than cross-sample
  RT alignment — use the upstream IDSL.IPA peak detection/quality steps instead
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.IPA
  role: Implements the dynamic retention index marker approach that corrects retention
    time and cross-references peaks from multiple samples; produces the per-sample
    peaklists fed into the correction
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: 'Runtime environment: IDSL.IPA is an R package (CRAN) whose IPA_workflow function
    is driven by a parameter spreadsheet to run peak detection, RT correction and
    alignment'
  repo: https://cran.r-project.org/package=IDSL.IPA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:09966fcc9b19cf12ec5dba113ac7828481bf48f20b721104538efc4e947556d6
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
yes_rate: 0.8
n_claims: 10
---

# lc-hrms-batch-correction-analysis

## Summary

Retention time (batch) correction for untargeted LC/HRMS peaklists using IDSL.IPA's dynamic retention index marker approach, which aligns per-sample m/z–RT pairs against a pool of recurring indigenous reference peaks rather than external standards.

## When to use

Use when aligning per-sample IDSL.IPA peaklists (m/z–RT pairs from LC/HRMS files) across a population-scale study where systematic retention time drift between batches or samples must be removed before peak cross-referencing.

## When NOT to use

- Input is already an aligned, retention-time-corrected feature table — re-applying the index correction would re-drift the aligned RTs
- Samples lack recurring endogenous reference peaks (few overlapping m/z–RT markers), forcing reliance on the median-drift fallback with weak correction power
- The goal is chromatographic peak detection or quality filtering rather than cross-sample RT alignment — use the upstream IDSL.IPA peak detection/quality steps instead

## Inputs

- Per-sample IDSL.IPA peaklists (m/z–RT pairs per HRMS file, e.g. from mzML/mzXML/netCDF)
- Study design / batch metadata for population-scale LC/HRMS studies (e.g. MTBLS136, n=1649; ST000923 n=604; ST001237 n=1220)

## Outputs

- Corrected retention time columns for every sample in a single CSV table (aligned peak table)
- Retention-index-corrected peaklists ready for downstream annotation (e.g. IDSL.UFA molecular formula annotation)

## How to apply

Load the per-sample IDSL.IPA peaklists (m/z–RT pairs extracted from mzXML/mzML/netCDF HRMS files) for the study into R. Build a dynamic pool of indigenous (endogenous) reference markers by clustering m/z–RT pairs that recur across samples, then for each peak locate the nearest preceding and following reference markers and apply the retention-index equation (5) log-interpolation to estimate the systematic drift. Repeat the correction in several steps using the reference peaks to increase statistical significance; when a sample lacks sufficient local markers, fall back to the median drift of the reference marker pool. Write the corrected retention time columns for every sample into a single aligned CSV table, and compare against polynomial regression as an alternative correction model.

## Related tools

- **IDSL.IPA** (Implements the dynamic retention index marker approach that corrects retention time and cross-references peaks from multiple samples; produces the per-sample peaklists fed into the correction) — https://github.com/idslme/IDSL.IPA
- **R** (Runtime environment: IDSL.IPA is an R package (CRAN) whose IPA_workflow function is driven by a parameter spreadsheet to run peak detection, RT correction and alignment) — https://cran.r-project.org/package=IDSL.IPA

## Examples

```
library(IDSL.IPA); IPA_workflow("Address of the IPA parameter spreadsheet")
```

## Evaluation signals

- Corrected retention times reduce the systematic RT drift across samples compared with uncorrected RTs (as reported for MTBLS136 with 1649 samples)
- Benchmark the retention-index correction against polynomial regression RT correction and confirm comparable or lower residual RT variation
- Each corrected RT is bracketed by identified preceding/following reference markers; flag samples where the median-drift fallback was used
- Aligned output CSV contains one corrected RT column per sample and every peak appears in exactly one aligned row
- Downstream IDSL.UFA annotation on the corrected peaklists consumes the peak_alignment directory without RT-mismatch errors

## Limitations

- Samples with insufficient local indigenous reference markers fall back to the pool-median drift, reducing correction accuracy for those samples
- Requires recurring m/z–RT marker pairs across samples, so very sparse or highly heterogeneous studies may yield a weak reference marker pool
- IDSL.IPA aligned data matrices still need further processing (adduct/fragment identification, statistics filtering, molecular formula annotation) after RT correction

## Evidence
