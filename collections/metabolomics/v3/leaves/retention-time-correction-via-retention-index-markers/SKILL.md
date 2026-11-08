---
name: retention-time-correction-via-retention-index-markers
description: Use when an untargeted LC/HRMS dataset (mzML/mzXML from hundreds of samples
  or multiple batches) shows systematic retention-time drift that must be removed
  before peak alignment or peak annotation.
when_to_use_negative:
- Input data are already aligned peak tables with corrected retention times and only
  need downstream formula annotation or statistics filtering
- LC/HRMS run lacks endogenous reference markers and only low-quality or non-reproducible
  peaks exist, so marker anchoring is unreliable
- The analysis goal is isotopologue pairing or peak-quality filtering rather than
  removing retention-time drift across samples or batches
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IDSL.IPA
  role: R package whose IPA_workflow implements the dynamic retention index marker
    retention-time correction across multiple batches before peak alignment
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: R
  role: Runtime environment used to execute the IDSL.IPA parallel retention-time correction
    and peak alignment steps
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: IDSL.UFA
  role: Downstream tool that consumes the aligned peak table and peaklists (with corrected
    RT) produced by IDSL.IPA for molecular formula annotation
  repo: https://github.com/idslme/IDSL.UFA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:4b1a4bf137492af2bc0d3fa985d5c73cc43c1ddbb8df39c14283555d645a43e8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - MS-DIAL
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Retention-time correction via retention-index markers

## Summary

A dynamic retention-index-marker approach for correcting systematic retention-time drift across LC/HRMS samples and batches by anchoring peak shifts to endogenous reference markers before cross-referencing peaks from multiple samples in IDSL.IPA.

## When to use

Use when an untargeted LC/HRMS dataset (mzML/mzXML from hundreds of samples or multiple batches) shows systematic retention-time drift that must be removed before peak alignment or peak annotation.

## When NOT to use

- Input data are already aligned peak tables with corrected retention times and only need downstream formula annotation or statistics filtering
- LC/HRMS run lacks endogenous reference markers and only low-quality or non-reproducible peaks exist, so marker anchoring is unreliable
- The analysis goal is isotopologue pairing or peak-quality filtering rather than removing retention-time drift across samples or batches

## Inputs

- Detected and characterized chromatographic peaks from untargeted LC/HRMS raw data (mzML, mzXML, netCDF) processed by IDSL.IPA
- Endogenous retention-index marker peaks present across samples
- IPA parameter spreadsheet specifying input/output locations and number of processing threads

## Outputs

- Retention-time-corrected peak lists per sample
- Aligned peak table (aligned peak intensity matrix with corrected m/z and RT) suitable for cross-referencing and annotation

## How to apply

Within the IDSL.IPA workflow, after peaks have been detected and characterized on extracted ion chromatograms, apply the dynamic retention index marker approach using endogenous reference markers: identify marker peaks present across samples, quantify the drift of sample peak retention times relative to these markers, and correct sample-specific retention times accordingly, which can be combined with or compared against polynomial regression methods. The article reports that for the MTBLS136 study with 1649 samples this approach removed the systematic retention time drift. Once corrected, the aligned retention times allow peaks from multiple samples to be cross-referenced during parallel peak alignment (e.g., the 20-thread alignment producing ~214,000 aligned peaks for 499 cord-blood samples in MTBLS1684). The corrected and aligned peak table (m/z, corrected RT, sample intensities) can then feed downstream annotation such as IDSL.UFA molecular formula assignment or annotation against a reference database of m/z and retention time. Parameters (data location, number of processing threads) are set in the IPA parameter spreadsheet that is fed to the IPA_workflow function.

## Related tools

- **IDSL.IPA** (R package whose IPA_workflow implements the dynamic retention index marker retention-time correction across multiple batches before peak alignment) — https://cran.r-project.org/package=IDSL.IPA
- **R** (Runtime environment used to execute the IDSL.IPA parallel retention-time correction and peak alignment steps) — https://cran.r-project.org/package=IDSL.IPA
- **IDSL.UFA** (Downstream tool that consumes the aligned peak table and peaklists (with corrected RT) produced by IDSL.IPA for molecular formula annotation) — https://github.com/idslme/IDSL.UFA

## Examples

```
library(IDSL.IPA); IPA_workflow("IPA_parameters.xlsx")
```

## Evaluation signals

- Plot corrected versus uncorrected retention times (or variation after retention time correction using the retention index approach and polynomial regression methods) to confirm systematic drift has been removed, as demonstrated for the MTBLS136 study with 1649 samples
- After correction, peaks shared across samples should align within the retention-time tolerance used by the alignment step — e.g., the 499-sample MTBLS1684 run should yield the reported ~214,000 aligned peaks with 82% (41,172 peak) overlap against MS-DIAL
- Peak quality and isotopologue-pairing outputs should be unchanged: corrected RTs must not alter S/N or peak-quality metrics that were applied before RT correction
- The aligned peak table must be a CSV containing per-sample intensity columns plus aligned m/z and corrected RT columns, and should be re-generable from the same IPA parameter spreadsheet and thread count (e.g., 20 threads)
- Downstream IDSL.UFA formula annotation should only accept peaklists and the peak_alignment directory generated by IDSL.IPA, confirming RT correction and alignment completed successfully

## Limitations

- IDSL.IPA aligned data matrices still require further processing for identifying adducts and fragments, filtering for statistics, and annotating molecular formulas by additional computational tools
- A majority of rejected EICs had low abundance but passed the S/N threshold; additional rules are needed to automatically filter out these low-abundance peaks, and such rejects also constrain which markers are available
- Performance depends on the chosen endogenous retention-index markers; the method was demonstrated with these markers in the IDSL.IPA pipeline and compared against polynomial regression, but marker selection rules are not exhaustively validated here

## Evidence
