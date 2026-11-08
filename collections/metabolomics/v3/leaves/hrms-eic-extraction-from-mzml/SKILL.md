---
name: hrms-eic-extraction-from-mzml
description: Use when you have raw HRMS peak data in mzML plus a validated compound
  list (CSV) of target or suspect masses, and you need EICs, MS2 scans, and per-peak
  quality flags before full identification.
when_to_use_negative:
- The input is already a processed feature table or peak list rather than raw vendor-independent
  mzML data — no extraction from raw scans is needed.
- You need untargeted peak detection across the full feature space (not just preselected
  compound/mass lists); tools like xcms, RforMassSpectrometry or patRoon are designed
  for that, whereas this skill extracts only listed masses.
- You need full compound identification rather than prescreening/QC — this skill only
  flags quality and selects representative MS2 spectra for downstream tools such as
  MetFrag.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MSnbase
  role: Modern low-level R package used to read the mzML file and retrieve precursor
    ion masses, EICs and MS2 spectral scans for the compound list
- name: mzR
  role: Low-level R alternative for flexible MS data access when parsing mzML during
    extraction
- name: Shinyscreen
  role: R package / Shiny application that packages this extraction-and-prescreening
    step with quality assessment and manual overrides
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: R
  role: Execution environment for the extraction, quality assessment and CSV summary
    generation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:160da53dcceb0d1d4ad3d3a8b4e45f8ef2890cc57361fef3d2d5f5e632e554f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - R
  - Shinyscreen
  - mzR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# hrms-eic-extraction-from-mzml

## Summary

Extract MS1 extracted ion chromatograms (EICs) and corresponding MS2 spectral scans for a preselected list of compounds or masses from a high-resolution mass spectrometry mzML file using MSnbase/mzR, then apply threshold-based quality assessment per MS1 parent data point. This is the extraction and prescreening stage of vendor-independent first-look HRMS inspection workflows such as Shinyscreen.

## When to use

Use when you have raw HRMS peak data in mzML plus a validated compound list (CSV) of target or suspect masses, and you need EICs, MS2 scans, and per-peak quality flags before full identification.

## When NOT to use

- The input is already a processed feature table or peak list rather than raw vendor-independent mzML data — no extraction from raw scans is needed.
- You need untargeted peak detection across the full feature space (not just preselected compound/mass lists); tools like xcms, RforMassSpectrometry or patRoon are designed for that, whereas this skill extracts only listed masses.
- You need full compound identification rather than prescreening/QC — this skill only flags quality and selects representative MS2 spectra for downstream tools such as MetFrag.

## Inputs

- HRMS raw data file in mzML format
- Validated compound list (CSV) with precursor structures/adduct settings or a mass list
- User-set experimental m/z error and retention time error tolerances
- QC thresholds: MS1 intensity threshold, MS2 intensity threshold, retention time shift limit

## Outputs

- CSV summary table with m/z, retention time, MS1/MS2 existence flags, noise level, retention-time-shift QC result, and selected representative MS2
- Extracted MS1 EICs within the specified m/z tolerance
- Corresponding MS2 spectral scans per input mass
- Saved extracted MS1/MS2 data object (e.g. R object) for plotting or downstream analysis

## How to apply

Parametrise the run with (a) experimental m/z and retention time errors, (b) quality control parameters such as MS1 and MS2 intensity thresholds and retention time shift, loading the compound list from CSV and the HRMS data from mzML. Read the mzML file with MSnbase (or the lower-level mzR) and, for each entry, automatically calculate the required precursor ion mass from the structural information using the given adduct setting. Retrieve the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance), and the corresponding MS2 spectral scans. Subject the extracted data to a quality assessment stage in which each MS1 parent data point is assessed based on the existence of MS1 and MS2 data, the noise level, and the retention time shift against the supplied thresholds. Use this step to select a representative MS2 spectrum (if present) for each input mass, flagging entries that fail threshold checks rather than dropping them. Write quality flags, selected representative MS2, retention times and intensities to a CSV summary table and persist the extracted MS1/MS2 data object for downstream visualisation or MetFrag integration.

## Related tools

- **MSnbase** (Modern low-level R package used to read the mzML file and retrieve precursor ion masses, EICs and MS2 spectral scans for the compound list)
- **mzR** (Low-level R alternative for flexible MS data access when parsing mzML during extraction)
- **Shinyscreen** (R package / Shiny application that packages this extraction-and-prescreening step with quality assessment and manual overrides) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **R** (Execution environment for the extraction, quality assessment and CSV summary generation)

## Examples

```
Rscript -e 'library(MSnbase); f <- readMSData("run.mzML", mode="onDisk"); eics <- chromatogram(f, mz=pmppm(118.0863, ppm=5))'
```

## Evaluation signals

- Summary CSV exists and contains the expected columns (m/z, retention time, quality result).
- Every compound/mass entry in the summary has a recorded MS1/MS2 existence flag plus a noise and retention-time-shift QC result.
- Entries failing the MS1/MS2 intensity or RT-shift threshold checks are flagged rather than silently dropped.
- Retrieved EICs fall within the user-specified m/z tolerance around the calculated precursor ion masses (adduct-aware).
- A representative MS2 spectrum is recorded for an entry only when MS2 data actually exists for that mass.

## Limitations

- Extraction is list-based only (compound list or mass list); it does not perform untargeted feature discovery across all detected peaks.
- Using the underlying low-level packages (mzR, MSnbase) directly requires programming expertise to fully utilise.
- QC is threshold-driven (MS1/MS2 intensity, noise, RT shift); results depend on sensible user-supplied tolerances and thresholds.
- No archived code snapshot (Zenodo/OSF) is reported for Shinyscreen in the source; only the GitLab repository and Apache 2.0 license are cited.

## Evidence

- [methods] the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance) as well as their corresponding MS2 spectral scans are retrieved from the: "the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance) as well as their corresponding MS2 spectral scans"
- [methods] The quality of each MS1 parent data point is assessed based on the existence of MS1 and MS2 data, the noise level and the retention time shift: "The quality of each MS1 parent data point is assessed based on the existence of MS1 and MS2 data, the noise level and the retention time shift"
- [intro] the more modern MSnbase [10, 11], which offer great flexibility in analysis of MS data: "powerful, but low-level R packages such as mzR [9] and the more modern MSnbase [10, 11], which offer great flexibility in analysis of MS data"
- [methods] Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting: "Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting"
- [methods] parametrise (a) experimental m/z and retention time errors: "parametrise (a) experimental m/z and retention time errors"
- [methods] quality control parameters such as MS1 and MS2 intensity thresholds and retention time shifts: "quality control parameters such as MS1 and MS2 intensity thresholds and retention time shifts"
