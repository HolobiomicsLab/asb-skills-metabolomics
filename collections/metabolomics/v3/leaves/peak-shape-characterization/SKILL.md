---
name: peak-shape-characterization
description: Use when processing raw untargeted LC/HRMS data (mzXML, mzML, netCDF)
  and each detected extracted ion chromatogram peak must be characterized by shape
  metrics (nIsoPair, RCS, RPW, S/N) to decide whether it passes quality thresholds.
when_to_use_negative:
- When the input is already a processed peak/feature table rather than raw LC/HRMS
  data — this skill operates on detected EIC peaks from raw files, not on prior feature
  matrices.
- When the goal is molecular formula annotation rather than peak characterization
  — use IDSL.UFA or IDSL.UFAx on the IDSL.IPA peaklists instead.
- When MS/MS spectra (DDA, DIA, MSE, AIF, SWATH) must be deconvoluted — IDSL.CSA covers
  that, not this peak-shape skill.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: IDSL.IPA
  role: R package that implements the peak detection, peak smoothing, baseline development
    for local noise measurement, and peak quality determination used to compute the
    19 peak properties and filter peaks.
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Runtime environment in which the IDSL.IPA package is executed (installed from
    CRAN).
  repo: https://cran.r-project.org/package=IDSL.IPA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:6f4ab00f18cc8d1d5dcc56095ab050ab4225fbf714841957f83b2efbf915beb2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# peak-shape-characterization

## Summary

Computes 19 chromatographic peak properties (peak area, nIsoPair, RCS, R13C, RPW, peak width, asymmetry, skewness, S/N, sharpness) for each detected EIC peak in IDSL.IPA to judge peak quality and reliability before a peak enters the aligned data matrix.

## When to use

Use when processing raw untargeted LC/HRMS data (mzXML, mzML, netCDF) and each detected extracted ion chromatogram peak must be characterized by shape metrics (nIsoPair, RCS, RPW, S/N) to decide whether it passes quality thresholds.

## When NOT to use

- When the input is already a processed peak/feature table rather than raw LC/HRMS data — this skill operates on detected EIC peaks from raw files, not on prior feature matrices.
- When the goal is molecular formula annotation rather than peak characterization — use IDSL.UFA or IDSL.UFAx on the IDSL.IPA peaklists instead.
- When MS/MS spectra (DDA, DIA, MSE, AIF, SWATH) must be deconvoluted — IDSL.CSA covers that, not this peak-shape skill.

## Inputs

- MS1 level untargeted LC/HRMS files in mzXML, mzML, or netCDF/CDF format (e.g. Thermo Q Exactive HF Orbitrap data from study ST002263, or file 003.d from MTBLS1684)
- IPA parameter spreadsheet (IPA_parameters.xlsx) with thresholds for peak quality criteria (nIsoPair, RCS, RPW, R13C, S/N using baseline, minimum peak height, chromatogram gap)
- Candidate m/z values from the 12C/13C ion pairing step applied to each mass spectrum

## Outputs

- Individual peaklist per HRMS file (.Rdata and .csv) in the 'peaklists' directory, with 19 chromatographic peak properties per peak
- Aligned peak tables in the 'peak_alignment' directory (downstream)
- Optional untargeted extracted ion chromatograms (EICs) in the 'IPA_EIC' directory

## How to apply

After IDSL.IPA has generated an EIC candidate for each candidate m/z from 12C/13C ion pairing in individual mass spectra, fit a smoothed peak and a local-noise baseline, then compute the peak properties defined in the package: peak area, cumulated intensity, nIsoPair, RCS, R13C, peak width, RPW, number of separation trays, asymmetry factor, USP tailing factor, skewness (derivative and pseudo-moment methods), symmetry, gaussianity, S/N using baseline, S/N using the xcms method, S/N using the RMS method, and sharpness. Configure these via the IPA parameter spreadsheet (IPA_parameters.xlsx) which drives the IPA_workflow function; each detected peak is then evaluated against the peak quality thresholds so that only peaks passing the IDSL.IPA isotopologue pairing and peak quality thresholds are retained in the individual peaklist. Peaks that pass are written to the per-file peaklist (.Rdata and .csv in the 'peaklists' directory), while low-abundance EICs that fail quality checks are rejected.

## Related tools

- **IDSL.IPA** (R package that implements the peak detection, peak smoothing, baseline development for local noise measurement, and peak quality determination used to compute the 19 peak properties and filter peaks.) — https://github.com/idslme/IDSL.IPA
- **R** (Runtime environment in which the IDSL.IPA package is executed (installed from CRAN).) — https://cran.r-project.org/package=IDSL.IPA

## Examples

```
library(IDSL.IPA); IPA_workflow("Address of the IPA parameter spreadsheet")
```

## Evaluation signals

- All peaks in the final individual peaklist satisfy the configured quality thresholds (minimum peak height, chromatogram gap %, nIsoPair, RCS, R13C, RPW, S/N using baseline) — peaks failing any of the seven criteria must be absent.
- Each retained peak carries all 19 peak properties (peak area, nIsoPair, RCS, cumulated intensity, R13C, peak width, RPW, number of separation trays, asymmetry factor, USP tailing factor, skewness/symmetry/gaussianity, three S/N variants, sharpness).
- Peak count is consistent with known behavior: on a single test file (003.d, MTBLS1684), IDSL.IPA detected 20,253 total peaks, and redundant 13C isotope peaks were removed so only the most abundant monoisotopic isotopologues remain.
- EICs rejected by the isotopologue pairing and peak quality thresholds are excluded from the output peaklist.
- Output files exist as expected: .Rdata and .csv per file under 'peaklists', and aligned tables under 'peak_alignment' when running the full workflow.

## Limitations

- Low-abundance EICs may pass the S/N ratio threshold yet be rejected; the authors note there is a need to develop additional rules to automatically filter out these rejected low abundant peaks.
- IDSL.IPA aligned data matrices need to be further processed for identifying adducts and fragments, filtering for statistics and annotating molecular formulas by additional computational tools.
- The pipeline specifically targets carbon-containing compounds via 12C/13C ion pairing, so inorganic or non-carbon analytes are not covered.

## Evidence

- [readme] 19 peak properties computed and used for quality determination: [evidence span withheld — anchor status: not_found]
- [results] Rejected EICs and the need for further automatic filtering rules: [evidence span withheld — anchor status: not_found]
