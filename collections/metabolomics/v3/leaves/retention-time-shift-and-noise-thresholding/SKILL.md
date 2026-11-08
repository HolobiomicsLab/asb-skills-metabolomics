---
name: retention-time-shift-and-noise-thresholding
description: Use when a targeted or suspect HRMS prescreening run has produced EICs
  and MS2 scans per compound and you must pass/fail each parent point against experimental
  m/z error, RT error, and MS1/MS2 intensity thresholds before further identification.
when_to_use_negative:
- When the task is de novo feature detection or peak picking across the full sample
  (e.g. untargeted discovery without a compound or mass list) — Shinyscreen's focus
  is exclusively on a preselected list of compounds or masses of interest, not general
  peak detection.
- When the goal is database-independent compound identification itself — this skill
  only prescreens and flags peaks; identification is delegated to tools such as MetFrag.
- When input data are already vendor-curated and quality-assessed (e.g. Xcalibur-processed
  results) — this skill targets raw vendor-independent HRMS data requiring automated
  QC.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Shinyscreen
  role: Runs the prescreening pipeline including the quality assessment stage that
    applies MS1/MS2 intensity thresholds, noise-level checks, and retention-time-shift
    checks to each MS1 parent data point.
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: MSnbase
  role: Low-level R package used to read the mzML file and retrieve precursor ion
    masses from the compound list, their EICs within the specified m/z tolerance,
    and the corresponding MS2 spectral scans that feed the thresholding stage.
- name: mzR
  role: Low-level R package offering flexible MS data access used for first-look data
    exploration and prescreening of HRMS data in the Shinyscreen ecosystem.
- name: MetFrag
  role: Downstream identification tool that consumes the prescreened, quality-checked
    representative MS2 spectra; Shinyscreen hides complexity and runs several MetFrag
    instances in background.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:b7925dbc6347cc8efcdeb3da916ca36879340400fa5eb5647be5f2bb46231e5c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - MetFrag
  - R
  - Shinyscreen
  - mzR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# retention-time-shift-and-noise-thresholding

## Summary

A QC filtering stage from Shinyscreen in which each extracted MS1 parent data point is judged by the existence of MS1 and MS2 data, the noise level, and the retention time shift, with failures flagged via MS1/MS2 intensity thresholds and retention-time-shift tolerance rather than silently dropped.

## When to use

Use when a targeted or suspect HRMS prescreening run has produced EICs and MS2 scans per compound and you must pass/fail each parent point against experimental m/z error, RT error, and MS1/MS2 intensity thresholds before further identification.

## When NOT to use

- When the task is de novo feature detection or peak picking across the full sample (e.g. untargeted discovery without a compound or mass list) — Shinyscreen's focus is exclusively on a preselected list of compounds or masses of interest, not general peak detection.
- When the goal is database-independent compound identification itself — this skill only prescreens and flags peaks; identification is delegated to tools such as MetFrag.
- When input data are already vendor-curated and quality-assessed (e.g. Xcalibur-processed results) — this skill targets raw vendor-independent HRMS data requiring automated QC.

## Inputs

- Validated compound list (CSV) with structural information and adduct settings
- HRMS raw data file in mzML format
- Extraction parameters: experimental m/z error, retention time error tolerance
- QC thresholds: MS1 intensity threshold, MS2 intensity threshold, maximum retention time shift

## Outputs

- Per-peak QC verdicts (quality result for each MS1 parent data point: existence of MS1 and MS2 data, noise level, retention time shift)
- CSV summary table with m/z, retention time, quality flags, intensities
- Selected representative MS2 spectrum per input mass
- Extracted MS1/MS2 data object saved for downstream use
- PDF report of MS1 and MS2 chromatograms coupled with MS2 spectra

## How to apply

After loading the validated compound list (CSV) and an mzML file, set the extraction tolerances — experimental m/z error and retention time error — plus QC thresholds (MS1 intensity threshold, MS2 intensity threshold, retention time shift). Shinyscreen calculates the required precursor ion masses from the given adduct setting, retrieves each precursor's EIC within the specified m/z tolerance and its corresponding MS2 spectral scans, then subjects the data to the quality assessment stage: the quality of each MS1 parent data point is assessed based on the existence of MS1 and MS2 data, the noise level and the retention time shift. Results that pass can also be filtered by quantities of interest such as m/z, retention time, or quality score. Write QC flags, selected representative MS2 spectra, retention times and intensities to a CSV summary and save the extracted MS1/MS2 data object so downstream identification or upload to public data resources operates only on quality-passing peaks.

## Related tools

- **Shinyscreen** (Runs the prescreening pipeline including the quality assessment stage that applies MS1/MS2 intensity thresholds, noise-level checks, and retention-time-shift checks to each MS1 parent data point.) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **MSnbase** (Low-level R package used to read the mzML file and retrieve precursor ion masses from the compound list, their EICs within the specified m/z tolerance, and the corresponding MS2 spectral scans that feed the thresholding stage.)
- **mzR** (Low-level R package offering flexible MS data access used for first-look data exploration and prescreening of HRMS data in the Shinyscreen ecosystem.)
- **MetFrag** (Downstream identification tool that consumes the prescreened, quality-checked representative MS2 spectra; Shinyscreen hides complexity and runs several MetFrag instances in background.)

## Examples

```
Rscript -e 'shinyscreen::prescreen(compounds="compounds.csv", mzml="sample.mzML", mz_tol=0.005, rt_tol=30, ms1_int_thr=1e4, ms2_int_thr=5e3, rt_shift_thr=10)'
```

## Evaluation signals

- The summary CSV exists and contains the expected columns (m/z, retention time, quality result).
- Every compound entry has a recorded MS1/MS2 existence flag, noise-level assessment, and retention-time-shift QC result.
- Entries failing MS1/MS2 intensity thresholds or exceeding the retention time shift are flagged rather than silently dropped.
- A representative MS2 spectrum is selected (if present) for each input mass, and the extracted MS1/MS2 data object is saved alongside the CSV.
- Downstream identification inputs (e.g. MetFrag submissions) contain only peaks that passed the quality assessment stage.

## Limitations

- Thresholds are user-supplied and data-dependent: choosing too strict an MS1/MS2 intensity threshold or too narrow an m/z/RT tolerance can flag legitimate low-abundance peaks; too lenient a threshold passes noise.
- The quality assessment requires existence of MS1 and MS2 data, so compounds with no recorded MS2 scan for the given adduct cannot produce a selected representative MS2 spectrum.
- Because the screening is list-based (compound list or mass list), it does not address peaks outside the supplied list — general untargeted feature detection is out of scope.
- Retention-time-shift checks assume the compound's expected RT window; large chromatographic drifts between runs may require re-tuning the RT error parameter before thresholds are applied.

## Evidence

- [methods] quality assessment stage: "the data is subjected to a quality assessment stage. The quality of each MS1 parent data point is assessed based on the existence of MS1 and MS2 data, the noise level and the retention time shift"
- [methods] QC thresholds parametrisation: [evidence span withheld — anchor status: not_found]
- [methods] filtering by quantities of interest: "It is also possible to filter based on quantities of interest such as m/z, retention time, or quality score"
- [abstract] vendor-independent prescreening niche: "As a vendor-independent, cross operating system application it covers an important niche in open mass spectrometry workflows."
- [abstract] list-based screening scope: "Its versatile list-based approach supports the curation of data starting from either known or "suspected" compounds (compound list-based screening) or detected masses (mass list-based screening)"
