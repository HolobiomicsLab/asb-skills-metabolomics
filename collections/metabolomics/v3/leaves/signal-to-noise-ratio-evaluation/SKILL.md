---
name: signal-to-noise-ratio-evaluation
description: Use when filtering detected chromatographic peaks in LC/HRMS peaklists
  by local-noise S/N, e.g. before peak alignment or downstream formula annotation.
when_to_use_negative:
- Input is raw spectra before peak detection — S/N filtering here operates on detected
  EIC peak candidates, not raw scans.
- The goal is de novo formula annotation or fragment/adduct assignment; use IDSL.UFA/UFAx
  or IDSL.CSA instead, which consume already-filtered IPA peaklists.
- Working in MS/MS (DDA/DIA) fragmentation mode — the S/N filter applies to MS1-level
  full-scan LC/HRMS peaks processed by IDSL.IPA.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IDSL.IPA
  role: Computes S/N using baseline, xcms, and RMS methods as peak quality criteria
    during peak property evaluation and peaklist generation
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Runtime environment; IDSL.IPA is an R package installed from CRAN and driven
    via the IPA_workflow function
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: xcms
  role: Alternative S/N definition ('S/N using the xcms method') available as a peak
    property in IDSL.IPA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:99d8ebc196148beaa1c35cc24874dec80f01cf195c0282211fc6760166fd919e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# signal-to-noise-ratio-evaluation

## Summary

Computes and applies signal-to-noise ratio (S/N) thresholds to distinguish true chromatographic peaks from baseline noise in untargeted LC/HRMS peaklists, using the baseline-based S/N method implemented in IDSL.IPA (with xcms- and RMS-method alternatives).

## When to use

Use when filtering detected chromatographic peaks in LC/HRMS peaklists by local-noise S/N, e.g. before peak alignment or downstream formula annotation.

## When NOT to use

- Input is raw spectra before peak detection — S/N filtering here operates on detected EIC peak candidates, not raw scans.
- The goal is de novo formula annotation or fragment/adduct assignment; use IDSL.UFA/UFAx or IDSL.CSA instead, which consume already-filtered IPA peaklists.
- Working in MS/MS (DDA/DIA) fragmentation mode — the S/N filter applies to MS1-level full-scan LC/HRMS peaks processed by IDSL.IPA.

## Inputs

- Extracted ion chromatograms (EICs) from untargeted LC/HRMS data (mzXML, mzML, netCDF)
- IPA parameter spreadsheet (IPA_parameters.xlsx) with S/N threshold parameters
- Detected chromatographic peak candidates with chromatographic properties

## Outputs

- Individual peaklists filtered by the S/N (and other) peak quality criteria, in .Rdata and .csv format in the peaklists directory
- Per-peak S/N values (using baseline, xcms, and RMS methods) recorded as peak properties

## How to apply

After peak detection on extracted ion chromatograms (EICs), develop a local baseline around the peak and calculate S/N using the baseline method as one of the peak quality criteria (PARAM sheet; alongside nIsoPair, RCS, R13C, RPW and chromatogram gap criteria). IDSL.IPA calculates S/N using three definitions — using baseline, using the xcms method, and using the RMS method — and the parameter spreadsheet lets you select the minimum acceptable S/N for peak retention. Apply the S/N threshold during the individual peaklist step so that only peaks passing all seven peak quality parameters are written to the standalone filtered peaklist. Note the article's finding that a majority of rejected EICs had low abundance but passed the S/N ratio threshold, so S/N alone is not sufficient and must be combined with the other quality filters.

## Related tools

- **IDSL.IPA** (Computes S/N using baseline, xcms, and RMS methods as peak quality criteria during peak property evaluation and peaklist generation) — https://github.com/idslme/IDSL.IPA
- **R** (Runtime environment; IDSL.IPA is an R package installed from CRAN and driven via the IPA_workflow function) — https://cran.r-project.org/package=IDSL.IPA
- **xcms** (Alternative S/N definition ('S/N using the xcms method') available as a peak property in IDSL.IPA)

## Examples

```
library(IDSL.IPA); IPA_workflow("Address of the IPA parameter spreadsheet")
```

## Evaluation signals

- Each retained peak in the individual peaklist meets the S/N threshold configured in the IPA parameter spreadsheet alongside the other seven peak quality criteria.
- Per-peak S/N values are present in the peaklist properties (S/N using baseline, S/N using the xcms method, S/N using the RMS method).
- Filtered peaklist row count is consistent with the expected retention fraction when compared to the unfiltered EIC candidate list.
- Downstream pipeline steps (alignment, IDSL.UFA annotation) receive only peaks that passed the S/N filter — no low-S/N peaks propagate.
- No standalone filtered table entries for EICs that were rejected despite passing S/N but failing other quality checks (nIsoPair, RCS, RPW, gap).

## Limitations

- A majority of the rejected EICs had low abundance but passed the S/N ratio threshold — S/N alone is insufficient and must be combined with other peak quality parameters (nIsoPair, RCS, R13C, RPW, gap criteria).
- The exact numeric default S/N threshold is not stated in the article; it must be read from the IPA parameter spreadsheet (PARAM values) and may need tuning per instrument/study.
- S/N method choice matters: baseline, xcms, and RMS definitions are all implemented and can produce different retention outcomes; select and document the chosen definition.
- Requires MS1-level full-scan HRMS data (mzXML, mzML, netCDF); netCDF additionally needs the RNetCDF package installed.

## Evidence

- [readme] S/N using baseline listed as one of 19 chromatographic peak properties: [evidence span withheld — anchor status: not_found]
- [other] Task workflow describing the seven peak quality thresholds including S/N using baseline: [evidence span withheld — anchor status: not_found]
- [results] Article notes low-abundance peaks passing S/N still rejected — S/N alone insufficient: [evidence span withheld — anchor status: not_found]
