---
name: isotopologue-pairing-assessment
description: Use when processing untargeted LC/HRMS mzXML/mzML/netCDF data with IDSL.IPA
  and you need to suppress redundant isotope/adduct-like peaks by pairing 12C and
  13C signals within each scan, or when troubleshooting why peaks were excluded by
  nIsoPair/RCS thresholds.
when_to_use_negative:
- When the goal is annotating already-filtered peaklists with molecular formulas —
  use IDSL.UFA or IDSL.UFAx instead of re-running ion pairing.
- When peak detection itself is the target rather than pre-detection isotope deconvolution;
  MZmine 2, xcms-CAMERA, and MS-DIAL detect chromatographic regions of interest without
  the scan-level 12C/13C pairing step.
- When processing netCDF/CDF data without installing RNetCDF — IDSL.IPA requires RNetCDF
  for netCDF/CDF inputs.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IDSL.IPA
  role: Implements the 12C/13C ion pairing and isotopologue-based peak filtering (nIsoPair,
    RCS, peak quality thresholds) within the IPA_workflow.
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Runtime language in which the IDSL.IPA package (installed from CRAN) executes
    ion pairing and peak detection.
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: IDSL.UFA
  role: Downstream formula annotation of the isotopologue-filtered peaklists produced
    by IDSL.IPA.
  repo: https://github.com/idslme/IDSL.UFA
- name: IDSL.UFAx
  role: Exhaustive elemental composition annotation of peaks from IDSL.IPA peaklists.
  repo: https://github.com/idslme/IDSL.UFAx
- name: IDSL.CSA
  role: Clusters recurring ions from IDSL.IPA output to generate composite spectra.
  repo: https://github.com/idslme/IDSL.CSA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:b3755dc29665308a365163e3d8f64c2db4f6d2873bd1415469fee7c00f9d70e6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.CSA
  - IDSL.IPA
  - IDSL.UFA
  - IDSL.UFAx
  - R
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# isotopologue-pairing-assessment

## Summary

Assess and apply 12C/13C isotopologue ion pairing in each mass spectrum to isolate high-quality, non-redundant carbon-containing peaks before EIC peak detection in IDSL.IPA, so that only the most abundant monoisotopic isotopologues are carried into peak analysis.

## When to use

Use when processing untargeted LC/HRMS mzXML/mzML/netCDF data with IDSL.IPA and you need to suppress redundant isotope/adduct-like peaks by pairing 12C and 13C signals within each scan, or when troubleshooting why peaks were excluded by nIsoPair/RCS thresholds.

## When NOT to use

- When the goal is annotating already-filtered peaklists with molecular formulas — use IDSL.UFA or IDSL.UFAx instead of re-running ion pairing.
- When peak detection itself is the target rather than pre-detection isotope deconvolution; MZmine 2, xcms-CAMERA, and MS-DIAL detect chromatographic regions of interest without the scan-level 12C/13C pairing step.
- When processing netCDF/CDF data without installing RNetCDF — IDSL.IPA requires RNetCDF for netCDF/CDF inputs.

## Inputs

- Untargeted LC/HRMS full-scan mass spectrometry data in mzXML, mzML, or netCDF/CDF format
- IPA parameter spreadsheet (IPA_parameters.xlsx) with ion pairing and peak quality settings
- Optional custom ion mass differences to screen in addition to natural 12C/13C isotopologues

## Outputs

- Candidate m/z value list from paired 12C/13C isotopologues per scan
- Filtered individual peaklists (Rdata and CSV) retaining only peaks passing isotopologue pairing and the seven peak quality criteria (nIsoPair, RCS, R13C, RPW, S/N, etc.)
- Aligned peak tables and optional untargeted EICs for subsequent formula annotation

## How to apply

In the IDSL.IPA pipeline (R package, run via IPA_workflow with the IPA parameter spreadsheet), mass spectra are scanned to pair potential 12C and 13C isotopologues in each individual scan; each 12C m/z peak is used in only one m/z cluster and is not used in overlapping clusters, so redundant isotope peaks are removed before peak detection. Only m/z values corresponding to the 12C isotopologue (the most abundant monoisotopic isotopologue) are then used for the extracted ion chromatogram (EIC) candidate generation and peak detection step. Parameters controlling ion pairing include the ion pairing mass difference screen (natural 12C/13C difference plus any custom ion mass differences in the spreadsheet) and the isotopologue pairing quality metrics nIsoPair and RCS, which later act as peak-quality filters: peaks that fail to pass the IDSL.IPA isotopologue pairing and peak quality thresholds are dropped from the individual peaklist (e.g. the 20,253 peaks detected for test file 003.d from MTBLS1684). The rationale is that a 12C/13C ion pairing approach in individual mass spectra minimizes redundant peaks prior to chromatographic peak detection, improving sensitivity, specificity and robustness for carbon-containing compounds in population-scale studies.

## Related tools

- **IDSL.IPA** (Implements the 12C/13C ion pairing and isotopologue-based peak filtering (nIsoPair, RCS, peak quality thresholds) within the IPA_workflow.) — https://github.com/idslme/IDSL.IPA
- **R** (Runtime language in which the IDSL.IPA package (installed from CRAN) executes ion pairing and peak detection.) — https://cran.r-project.org/package=IDSL.IPA
- **IDSL.UFA** (Downstream formula annotation of the isotopologue-filtered peaklists produced by IDSL.IPA.) — https://github.com/idslme/IDSL.UFA
- **IDSL.UFAx** (Exhaustive elemental composition annotation of peaks from IDSL.IPA peaklists.) — https://github.com/idslme/IDSL.UFAx
- **IDSL.CSA** (Clusters recurring ions from IDSL.IPA output to generate composite spectra.) — https://github.com/idslme/IDSL.CSA

## Examples

```
library(IDSL.IPA); IPA_workflow("Address of the IPA parameter spreadsheet")
```

## Evaluation signals

- Every retained peak in the filtered individual peaklist satisfies the seven peak quality thresholds including nIsoPair and RCS; peaks that failed the IDSL.IPA isotopologue pairing and peak quality thresholds are absent.
- Each individual 12C m/z peak appears in only one m/z cluster and is not used in overlapping m/z clusters.
- Only the most abundant monoisotopic (12C) isotopologues appear in the detected peaklist; redundant isotope peaks are removed.
- For the MTBLS1684 test file 003.d the pipeline detects 20,253 total peaks, giving a concrete reference count.
- Theoretical and experimental R13C agreement can be checked via Spearman correlation (p ≤ 2.2*10−16, ρ ≤ 0.92) for authentic standards, supporting the isotopologue quality metrics.

## Limitations

- Low-abundance EICs that passed the S/N ratio threshold may still be rejected; the authors note additional rules are needed to automatically filter out these rejected low abundant peaks.
- Aligned IDSL.IPA data matrices require further processing for adducts/fragments identification, statistics filtering, and molecular formula annotation by additional computational tools.
- The ion pairing approach had not previously been used for large-scale untargeted metabolomics data processing pipelines, so behavior outside the evaluated study sizes (200–1,600 samples) is less characterized.

## Evidence

- [results] Additional rules needed for rejected low abundant peaks: [evidence span withheld — anchor status: not_found]
