---
name: retention-time-drift-correction
description: Use when per-sample LC/HRMS peaklists (m/z–RT pairs) from a multi-sample
  or multi-batch study show systematic RT drift and peaks must be aligned before cross-referencing
  or building an intensity matrix.
when_to_use_negative:
- The data is already an aligned, RT-corrected feature/intensity matrix (the correction
  step has been performed upstream)
- Only a single LC/HRMS file is analyzed with no cross-sample or cross-batch comparison,
  so no inter-sample drift exists to correct
- The study contains too few recurring endogenous peaks to form a reference marker
  pool, leaving no anchors for interpolation
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: IDSL.IPA
  role: Implements the dynamic retention index marker approach for RT correction and
    cross-referencing peaks from multiple samples within the IPA_workflow pipeline
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Runtime environment in which the IDSL.IPA package's RT correction and alignment
    steps are executed
  repo: https://cran.r-project.org/package=IDSL.IPA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:bdabe22a0bcc75f997b4863fc08b3e921ef4015d2e5296cb92790a1c607466c4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - R
  - ncGTW
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# retention-time-drift-correction

## Summary

Corrects systematic retention time (RT) drift across LC/HRMS samples using IDSL.IPA's dynamic retention index marker approach, which anchors each peak to preceding and following indigenous reference peaks rather than external standards.

## When to use

Use when per-sample LC/HRMS peaklists (m/z–RT pairs) from a multi-sample or multi-batch study show systematic RT drift and peaks must be aligned before cross-referencing or building an intensity matrix.

## When NOT to use

- The data is already an aligned, RT-corrected feature/intensity matrix (the correction step has been performed upstream)
- Only a single LC/HRMS file is analyzed with no cross-sample or cross-batch comparison, so no inter-sample drift exists to correct
- The study contains too few recurring endogenous peaks to form a reference marker pool, leaving no anchors for interpolation

## Inputs

- Per-sample IDSL.IPA peaklists (m/z–RT pairs per HRMS file, e.g. mzML, mzXML, netCDF)
- IPA parameter spreadsheet used as input for the IPA_workflow function
- Aligned peak table / peak_alignment directory from an IDSL.IPA run on multiple samples

## Outputs

- Corrected retention time columns for every sample written into a single aligned CSV table
- Cross-referenced peaks indexed against the dynamic retention index marker pool

## How to apply

Load the per-sample IDSL.IPA peaklists (m/z–RT pairs extracted from raw files such as mzML/mzXML/netCDF) and build a dynamic pool of endogenous (indigenous) retention index markers by clustering m/z–RT pairs that recur across samples. For each peak, locate the nearest preceding and following reference markers and interpolate the systematic RT drift between them (equation 5 of the IDSL.IPA paper), rather than fitting a global polynomial. Repeat the correction in several successive steps using the reference peaks to increase statistical significance. If a sample lacks sufficient local markers, fall back to the median drift of the reference marker pool. The method was shown to remove systematic retention time drift for the MTBLS136 study with 1649 samples, and the supplementary material compares its variation against polynomial regression correction methods.

## Related tools

- **IDSL.IPA** (Implements the dynamic retention index marker approach for RT correction and cross-referencing peaks from multiple samples within the IPA_workflow pipeline) — https://github.com/idslme/IDSL.IPA
- **R** (Runtime environment in which the IDSL.IPA package's RT correction and alignment steps are executed) — https://cran.r-project.org/package=IDSL.IPA

## Examples

```
library(IDSL.IPA); IPA_workflow("IPA_parameters.xlsx")
```

## Evaluation signals

- Corrected retention times show reduced systematic RT drift across samples compared with the uncorrected RTs (as demonstrated for MTBLS136, n=1649)
- Corrected RT variation is comparable to or better than a polynomial regression RT correction baseline (supplementary comparison)
- The reference marker pool consists of m/z–RT pairs that recur across samples, and each peak interpolates between a preceding and a following marker
- The aligned output table preserves the original peak set with only RT columns adjusted, so peak m/z values and intensities are unchanged

## Limitations

- When a sample lacks sufficient local reference markers, the method falls back to the median drift of the reference marker pool, which may be less accurate than local interpolation
- The correction depends on the dynamic pool of indigenous reference peaks; it relies on endogenous markers rather than injected standards, so marker density determines local correction fidelity
- The supplementary material notes variation between retention index and polynomial regression correction methods, indicating the choice of RT correction strategy affects downstream corrected RT values

## Evidence
