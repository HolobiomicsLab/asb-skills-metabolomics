---
name: lc-hrms-peak-detection
description: Use when raw untargeted LC/HRMS data (mzXML, mzML, netCDF) from population-scale
  studies must be converted into a peak intensity matrix of organic compounds, and
  high sensitivity and specificity for carbon-containing peaks is required.
when_to_use_negative:
- Input is already a processed feature/peak intensity table or aligned data matrix
  — the pipeline operates on raw MS1-level HRMS files, and IDSL.IPA aligned data matrices
  'need to be further processed for identifying adducts and fragments, filtering for
  statistics and annotating molecular formulas' by separate tools (IDSL.UFA, IDSL.UFAx).
- Molecular formula or structural annotation is the goal rather than peak detection
  — use IDSL.UFA or IDSL.UFAx downstream of IDSL.IPA for MS1-only isotopic profile
  matching or exhaustive chemical enumeration.
- Data are MS/MS fragment spectra or DDA/DIA/SWATH acquisitions rather than full-scan
  MS1 data — IDSL.CSA is the companion package for deconvolution of fragmentation
  spectra.
- The sample set is small (a single file or <100 samples) and low sensitivity for
  carbon-containing compounds is acceptable — generic tools like MZmine 2, xcms-CAMERA
  or MS-DIAL may suffice, though IDSL.IPA still detects 1.6x–5.4x more peaks on the
  same file.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IDSL.IPA
  role: Core R package implementing the 12C/13C ion pairing, EIC peak detection, recursive
    mass correction, retention time correction, and parallel alignment workflow, driven
    by a parameter spreadsheet.
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: R
  role: Runtime environment; the pipeline is an R package with the IPA_workflow function
    as the entry point.
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: MS-DIAL
  role: Benchmark baseline tool for peak-count comparison on the same raw file (IDSL.IPA
    detected 2.2x more peaks on 003.d).
- name: MZmine 2
  role: Benchmark baseline tool for peak-count comparison on the same raw file (IDSL.IPA
    detected 1.6x more peaks on 003.d).
- name: xcms-CAMERA
  role: Benchmark baseline tool for peak-count comparison on the same raw file (IDSL.IPA
    detected 5.4x more peaks on 003.d).
- name: IDSL.UFA
  role: Downstream annotation package that consumes the IDSL.IPA peaklists and aligned
    peak table for MS1-only molecular formula assignment.
  repo: https://github.com/idslme/IDSL.UFA
- name: IDSL.CSA
  role: Companion package for downstream deconvolution of MS1-only composite spectra,
    DDA, and DIA (MSE/AIF/SWATH) fragmentation data.
  repo: https://github.com/idslme/IDSL.CSA
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:9b94f75c421d378c2a0ca904c51edfcf94367c4d203f5f030b65f350be5e3ecf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.CSA
  - IDSL.IPA
  - IDSL.UFA
  - MS-DIAL
  - MZmine
  - MZmine 2
  - R
  - enviPat
  - xcms-CAMERA
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# lc-hrms-peak-detection

## Summary

Detect and characterize chromatographic peaks in untargeted LC/HRMS data by pairing 12C/13C isotopologues in individual mass spectra, building EICs, and applying mass correction, peak smoothing, local-noise baselines, and peak-quality thresholds — the IDSL.IPA approach that outperforms MZmine 2, xcms-CAMERA and MS-DIAL in sensitivity and specificity.

## When to use

Use when raw untargeted LC/HRMS data (mzXML, mzML, netCDF) from population-scale studies must be converted into a peak intensity matrix of organic compounds, and high sensitivity and specificity for carbon-containing peaks is required.

## When NOT to use

- Input is already a processed feature/peak intensity table or aligned data matrix — the pipeline operates on raw MS1-level HRMS files, and IDSL.IPA aligned data matrices 'need to be further processed for identifying adducts and fragments, filtering for statistics and annotating molecular formulas' by separate tools (IDSL.UFA, IDSL.UFAx).
- Molecular formula or structural annotation is the goal rather than peak detection — use IDSL.UFA or IDSL.UFAx downstream of IDSL.IPA for MS1-only isotopic profile matching or exhaustive chemical enumeration.
- Data are MS/MS fragment spectra or DDA/DIA/SWATH acquisitions rather than full-scan MS1 data — IDSL.CSA is the companion package for deconvolution of fragmentation spectra.
- The sample set is small (a single file or <100 samples) and low sensitivity for carbon-containing compounds is acceptable — generic tools like MZmine 2, xcms-CAMERA or MS-DIAL may suffice, though IDSL.IPA still detects 1.6x–5.4x more peaks on the same file.

## Inputs

- MS1-level untargeted LC/HRMS raw data files (mzXML, mzML, netCDF), e.g. the 003.d test file from the MTBLS1684 study
- IPA parameter spreadsheet (IPA_parameters.xlsx) with configured input/output paths, thread count, and recursive mass correction parameters
- For alignment across samples: a batch of individual peaklists from a population-scale study (e.g. 499 cord blood samples from MTBLS1684)

## Outputs

- Individual peaklist tables (peaklist_003.csv) with per-peak m/z, RT, and chromatographic properties — 20,253 total peaks for file 003.d from MTBLS1684
- Aligned peak intensity matrix for multi-sample studies (~214,000 aligned peak rows for 499 MTBLS1684 samples)
- Count of false-positive peaks removed by recursive mass correction (2,679 of 23,740 peaks, ~11%, for file 003.d)

## How to apply

Load the raw MS1-level HRMS data (mzXML/mzML/netCDF) and configure the IPA parameter spreadsheet: set PARAM0007 to the HRMS data location, PARAM0010 to the output location, PARAM0006 to the number of processing threads, and PARAM0019–PARAM0020 for recursive mass correction. Run the IPA_workflow function from the IDSL.IPA R package, which first pairs 12C/13C isotopologues in each individual mass spectrum to generate candidate monoisotopic m/z values and removes redundant isotope peaks — only the m/z values corresponding to the 12C isotopologue are used for peak detection, and each 12C m/z peak is used in only one m/z cluster. Next, build extracted ion chromatograms and characterize chromatographic peaks using recursive mass correction (re-checking peaks against the chromatogram trace of the intensity-weighted average mass, m/z-bar), peak smoothing, and baseline development for local noise measurement; the recursive mass correction step removed 2,679 (11%) of 23,740 initially detected peaks as false positives on the MTBLS1684 test file 003.d. Retain only peaks passing the IDSL.IPA isotopologue-pairing and peak-quality thresholds (S/N ratio, peak width, asymmetry and related chromatographic properties), then, for multi-sample studies, correct retention time with the dynamic retention index marker approach and align peaks in parallel across m/z slices (the 499-sample MTBLS1684 study yielded ~214,000 aligned peaks within 1 hour on 20 threads).

## Related tools

- **IDSL.IPA** (Core R package implementing the 12C/13C ion pairing, EIC peak detection, recursive mass correction, retention time correction, and parallel alignment workflow, driven by a parameter spreadsheet.) — https://cran.r-project.org/package=IDSL.IPA
- **R** (Runtime environment; the pipeline is an R package with the IPA_workflow function as the entry point.) — https://cran.r-project.org/package=IDSL.IPA
- **MS-DIAL** (Benchmark baseline tool for peak-count comparison on the same raw file (IDSL.IPA detected 2.2x more peaks on 003.d).)
- **MZmine 2** (Benchmark baseline tool for peak-count comparison on the same raw file (IDSL.IPA detected 1.6x more peaks on 003.d).)
- **xcms-CAMERA** (Benchmark baseline tool for peak-count comparison on the same raw file (IDSL.IPA detected 5.4x more peaks on 003.d).)
- **IDSL.UFA** (Downstream annotation package that consumes the IDSL.IPA peaklists and aligned peak table for MS1-only molecular formula assignment.) — https://github.com/idslme/IDSL.UFA
- **IDSL.CSA** (Companion package for downstream deconvolution of MS1-only composite spectra, DDA, and DIA (MSE/AIF/SWATH) fragmentation data.) — https://github.com/idslme/IDSL.CSA

## Examples

```
Rscript -e 'install.packages("IDSL.IPA"); library(IDSL.IPA); IPA_workflow("IPA_parameters.xlsx")'
```

## Evaluation signals

- Peak count matches the reference: 20,253 total peaks for the single test file 003.d from MTBLS1684.
- Recursive mass correction discard rate matches: 2,679 of 23,740 initially detected peaks (~11%) removed as false positives on 003.d.
- Fold-ratio benchmark on 003.d holds: 1.6x vs MZmine 2, 5.4x vs xcms-CAMERA, 2.2x vs MS-DIAL.
- For population-scale alignment (e.g. 499 MTBLS1684 samples), the aligned table reaches ~214,000 peaks and 82% shared-peak overlap (41,172 peaks) with MS-DIAL within ~1 hour on 20 parallel threads.
- Theoretical vs experimental R13C agreement for authentic standards: Spearman correlation p-value ≤ 2.2*10−16 and ρ ≤ 0.92.

## Limitations

- IDSL.IPA aligned data matrices need further processing for adduct/fragment identification, statistical filtering, and molecular formula annotation by additional tools.
- A majority of rejected EICs had low abundance but passed the S/N ratio threshold; additional rules are needed to automatically filter these low-abundant peaks.
- No software version is stated in the article; parameter defaults and thresholds should be pinned when reproducing.
- Isotopologue pairing assumes carbon-containing compounds — peaks without detectable 12C/13C pairs may be missed by design.

## Evidence

- [abstract] Pipeline purpose: organic compounds in LC/HRMS: [evidence span withheld — anchor status: not_found]
- [readme] Benchmark: outperforms MZmine 2, xcms, MS-DIAL: [evidence span withheld — anchor status: external_doc]
