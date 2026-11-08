---
name: chromatographic-peak-quality-assessment
description: Use when you have targeted/suspect HRMS data (mzML) plus a compound or
  mass list and need per-peak QC (MS1/MS2 existence, noise, RT shift) before further
  identification or spectral upload.
when_to_use_negative:
- Input is already a processed feature table or peak matrix — this skill operates
  on raw mzML data for a preselected list of compounds or masses of interest
- The goal is untargeted peak detection/feature finding across all detected masses
  rather than QC of listed targets or suspects
- A complex, fully resource-intensive annotation platform is already being used —
  Shinyscreen deliberately occupies the niche between low-level programming tools
  and such platforms, not their replacement
tools:
- name: Shinyscreen
  role: R package/Shiny application implementing the quality-assessment stage (MS1/MS2
    existence, noise level, retention-time shift) over extracted peak data
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: MSnbase
  role: Reads the mzML file and retrieves precursor ion masses, EICs within the m/z
    tolerance, and corresponding MS2 spectral scans feeding the QC stage
- name: mzR
  role: Low-level R package alternative for MS data access underlying first-look prescreening
    of HRMS data
- name: R
  role: Runtime language in which the extraction, prescreening, and QC steps are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:783991cb6450f37ef35b73e6afbc7ece68672d4613fa660ab5c2118cb508939f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - R
  - Shinyscreen
  - mzR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# chromatographic-peak-quality-assessment

## Summary

Quality-assessment step of Shinyscreen's extraction/prescreening stage: each MS1 parent data point for compounds from a preselected list is scored for MS1/MS2 data existence, noise level, and retention-time shift against user-supplied intensity and shift thresholds before any identification attempt.

## When to use

Use when you have targeted/suspect HRMS data (mzML) plus a compound or mass list and need per-peak QC (MS1/MS2 existence, noise, RT shift) before further identification or spectral upload.

## When NOT to use

- Input is already a processed feature table or peak matrix — this skill operates on raw mzML data for a preselected list of compounds or masses of interest
- The goal is untargeted peak detection/feature finding across all detected masses rather than QC of listed targets or suspects
- A complex, fully resource-intensive annotation platform is already being used — Shinyscreen deliberately occupies the niche between low-level programming tools and such platforms, not their replacement

## Inputs

- Validated compound list (CSV) with precursor ion masses and adduct settings
- High-resolution mass spectrometry mzML file (e.g. the Saccharomyces cerevisiae amino acids teaching dataset)
- Extraction parameters: experimental m/z error and retention-time error tolerances
- QC thresholds: MS1 intensity threshold, MS2 intensity threshold, retention-time shift limit

## Outputs

- CSV summary table with quality flags, m/z, retention time, intensities, and recorded MS1/MS2 existence plus noise/RT-shift QC result per compound
- Selected representative MS2 spectrum for each input mass (only where present)
- Extracted MS1/MS2 data object for downstream visualisation and MetFrag integration

## How to apply

Read the mzML file with MSnbase and retrieve the precursor ion masses from the compound list, their extracted ion chromatograms (EICs) within the specified m/z tolerance, and the corresponding MS2 spectral scans. Parametrise experimental m/z and retention-time errors plus QC thresholds such as MS1 and MS2 intensity thresholds and retention-time shifts. Subject the extracted data to the quality assessment stage: for each MS1 parent data point, check the existence of MS1 and MS2 data, the noise level, and the retention-time shift against the supplied thresholds. Use the same stage to select a representative MS2 spectrum (if present) for each input mass. Entries failing threshold checks must be flagged rather than silently dropped, so the analyst retains manual overrides consistent with Shinyscreen's automated-QC-plus-override design.

## Related tools

- **Shinyscreen** (R package/Shiny application implementing the quality-assessment stage (MS1/MS2 existence, noise level, retention-time shift) over extracted peak data) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **MSnbase** (Reads the mzML file and retrieves precursor ion masses, EICs within the m/z tolerance, and corresponding MS2 spectral scans feeding the QC stage)
- **mzR** (Low-level R package alternative for MS data access underlying first-look prescreening of HRMS data)
- **R** (Runtime language in which the extraction, prescreening, and QC steps are executed)

## Evaluation signals

- The summary CSV exists and contains the expected columns (m/z, retention time, quality result)
- Every compound entry in the summary has a recorded MS1/MS2 existence flag and a noise/retention-time-shift QC result
- Entries failing the MS1/MS2 intensity or RT-shift threshold checks appear flagged in the output, never silently dropped
- A representative MS2 spectrum is recorded only for input masses where MS2 data exist
- EIC extraction falls within the configured m/z tolerance; no peaks outside the tolerance appear in the QC table

## Limitations

- mzR and MSnbase are low-level and require some programming expertise to fully utilize
- QC is centered on a preselected list of compounds or masses of interest; untargeted mass discovery is out of scope
- The stage provides automated quality control combined with manual overrides — it does not by itself perform compound identification (MetFrag integration is a separate step)
- License is stated (Apache 2.0) but no archived code snapshot (Zenodo/OSF) is reported

## Evidence

- [methods] Quality assessment stage checks: "the data is subjected to a quality assessment stage. The quality of each MS1 parent data point is assessed based on the existence of MS1 and MS2 data, the noise level and the retention time shift"
- [methods] QC thresholds: "quality control parameters such as MS1 and MS2 intensity thresholds and retention time shifts"
- [methods] Extraction inputs to QC: "the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance) as well as their corresponding MS2 spectral scans"
- [methods] Representative MS2 selection: "This step is also used to select a representative MS2 spectra (if present) for the input masses"
- [abstract] Overall prescreening purpose: "a fully open source prescreening application to assist analysts in the high throughput quality control of the thousands of peaks detected in high resolution mass spectrometry experiments"
- [intro] Underlying packages: "the more modern MSnbase [10, 11], which offer great flexibility in analysis of MS data"
