---
name: chromatographic-peak-quality-filtering
description: Use when processing MS1-level LC/HRMS peak candidates (e.g. from IDSL.IPA
  EIC detection) and you need to discard low-quality or redundant isotope peaks using
  the seven-parameter peak quality filter before building aligned data matrices.
when_to_use_negative:
- Input is already an aligned, filtered feature/peak intensity matrix — no per-peak
  quality re-evaluation is required at this stage
- You need molecular formula annotation rather than peak filtering — use IDSL.UFA
  or IDSL.UFAx instead, which consume IDSL.IPA peaklists
- Peak candidates come from tools other than IDSL.IPA and you cannot supply their
  chromatographic properties (nIsoPair, RCS, RPW, S/N) needed by the seven criteria
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: IDSL.IPA
  role: Implements the seven peak-quality parameter filter and recursive mass correction
    during peak analysis; run via IPA_workflow with a parameter spreadsheet
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: R
  role: Runtime environment for the IDSL.IPA package and the IPA_workflow call that
    executes the quality-filter step
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: enviPat
  role: Used for calculating theoretical isotopic profiles for molecular formulas,
    supporting the 12C/13C pairing logic underlying the nIsoPair/RCS thresholds
- name: MZmine 2
  role: Benchmark peak-detection tool whose peaks are re-scored against IDSL.IPA's
    quality thresholds in the comparison workflow
- name: MS-DIAL
  role: Benchmark tool; its detected peaks absent from the IDSL.IPA list are evaluated
    against the isotopologue-pairing and quality thresholds
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_008
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:360bf9bd2a09b321ad1c738f3641d9da149130096d8691cf30135f3fcf1861ba
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - MS-DIAL
  - MZmine
  - MZmine 2
  - R
  - enviPat
  - xcms-CAMERA
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# chromatographic-peak-quality-filtering

## Summary

A post-detection filter step from the IDSL.IPA pipeline that evaluates every chromatographic peak candidate against seven quality criteria (peak height, gap percentage, nIsoPair, RCS, R13C, RPW, and baseline S/N) and retains only peaks passing all thresholds, thereby isolating reliable signals for carbon-containing organic compounds in untargeted LC/HRMS data.

## When to use

Use when processing MS1-level LC/HRMS peak candidates (e.g. from IDSL.IPA EIC detection) and you need to discard low-quality or redundant isotope peaks using the seven-parameter peak quality filter before building aligned data matrices.

## When NOT to use

- Input is already an aligned, filtered feature/peak intensity matrix — no per-peak quality re-evaluation is required at this stage
- You need molecular formula annotation rather than peak filtering — use IDSL.UFA or IDSL.UFAx instead, which consume IDSL.IPA peaklists
- Peak candidates come from tools other than IDSL.IPA and you cannot supply their chromatographic properties (nIsoPair, RCS, RPW, S/N) needed by the seven criteria

## Inputs

- MS1-level LC/HRMS data file (mzXML, mzML, or netCDF format, e.g. 003.d from MTBLS1684)
- IPA parameter spreadsheet (IPA_parameters.xlsx) with the seven peak-quality thresholds (PARAM0019–PARAM0020 for recursive mass correction and quality settings)
- IDSL.IPA EIC peak candidate table with chromatographic properties (height, width, asymmetry, S/N)

## Outputs

- Filtered individual peaklist table (peaklist_003.csv style) containing only peaks passing all seven quality criteria
- Per-peak quality metrics (nIsoPair, RCS, R13C, RPW, S/N vs baseline) for auditability
- Benchmark comparison table (peak_count_comparison.csv) when thresholds are applied to peaks detected by other tools

## How to apply

After IDSL.IPA generates EIC peak candidates for a file (e.g. 003.d from MTBLS1684), load the candidate peak table plus the chromatographic properties computed by the pipeline (peak height, width, asymmetry, local baseline noise). Read the seven peak-quality parameter settings from the IPA parameter spreadsheet (IPA_parameters.xlsx). Evaluate each candidate against the seven criteria in sequence — minimum peak height, chromatogram gap percentage, nIsoPair (isotopologue pairing count), RCS (resolving coupled peaks / ratio consistency), R13C (12C/13C ratio), RPW (retention-time peak width ratio), and S/N computed against the developed local baseline — and retain only peaks that pass all of them. The rationale is that peaks failing isotopologue pairing or quality thresholds are redundant isotope peaks or noise: the pipeline removes redundant isotope peaks, detecting only the most abundant monoisotopic isotopologues from 12C/13C clusters. On the MTBLS1684 test file the pipeline ultimately reported 20,253 retained peaks (vs ~23,740 pre-correction), with recursive mass correction removing 2,679 (~11%) false positives; peaks failing these thresholds in benchmark tools (MZmine 2, xcms-CAMERA, MS-DIAL) were scored against the same criteria.

## Related tools

- **IDSL.IPA** (Implements the seven peak-quality parameter filter and recursive mass correction during peak analysis; run via IPA_workflow with a parameter spreadsheet) — https://cran.r-project.org/package=IDSL.IPA
- **R** (Runtime environment for the IDSL.IPA package and the IPA_workflow call that executes the quality-filter step) — https://cran.r-project.org/package=IDSL.IPA
- **enviPat** (Used for calculating theoretical isotopic profiles for molecular formulas, supporting the 12C/13C pairing logic underlying the nIsoPair/RCS thresholds)
- **MZmine 2** (Benchmark peak-detection tool whose peaks are re-scored against IDSL.IPA's quality thresholds in the comparison workflow)
- **MS-DIAL** (Benchmark tool; its detected peaks absent from the IDSL.IPA list are evaluated against the isotopologue-pairing and quality thresholds)

## Examples

```
library(IDSL.IPA); IPA_workflow("IPA_parameters.xlsx")
```

## Evaluation signals

- Retained peak count for a known test file matches the reported value (e.g. 20,253 peaks for 003.d from MTBLS1684)
- Recursive mass correction removes ~2,679 of ~23,740 peaks (~11%) on the same test file
- Every retained peak passes all seven criteria (minimum height, gap %, nIsoPair, RCS, R13C, RPW, baseline S/N) with no failing threshold silently accepted
- Only the most abundant monoisotopic 12C isotopologues are present — redundant isotope peaks are absent from the filtered list
- Cross-tool benchmark fold ratios reproduce the reported values: 1.6x vs MZmine 2, 5.4x vs xcms-CAMERA, 2.2x vs MS-DIAL

## Limitations

- Low-abundance peaks may pass the S/N ratio threshold yet still be rejected manually; the article notes additional rules are needed to automatically filter these low-abundant peaks
- IDSL.IPA aligned data matrices still require further processing (adduct/fragment identification, statistical filtering, molecular formula annotation) after this filtering step
- No specific IDSL.IPA software version is stated in the article, complicating exact reproduction
- Threshold values live in the parameter spreadsheet and depend on user configuration; the article does not enumerate numeric defaults for each of the seven criteria

## Evidence

- [abstract] Peak quality determination drives the filter: "detecting and characterizing chromatographic peaks using a new sensitive and versatile approach to perform mass correction, peak smoothing, baseline development for local noise measurement, and peak"
- [results] Only peaks passing isotopologue pairing and quality thresholds are retained: "failed to pass the IDSL.IPA isotopologue pairing and peak quality thresholds"
- [results] Redundant isotope peaks are removed before downstream steps: "IDSL.IPA detects only the most abundant monoisotopic isotopologues from 12C/13C isotope clusters and removes the redundant isotope peaks"
- [results] Test-file benchmark supports expected filter outcome: "IDSL.IPA detected 20,253 total peaks for a single test LC/HRMS file (003.d) from the MTBLS1684 study"
- [supplementary] Recursive mass correction false-positive removal statistic: "Distribution of m/z changes after using an intensity weighted average mass from test LC/MS file (003.d)"
- [readme] README confirms peak-property evaluation and recursive mass correction as pipeline components: "IDSL.IPA is a suite of new algorithms covering extracted ion chromatogram (EIC) candidate generation, peak detection, peak property evaluation, recursive mass correction, retention time correction"
