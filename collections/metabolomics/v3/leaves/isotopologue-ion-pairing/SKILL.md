---
name: isotopologue-ion-pairing
description: Use when processing untargeted LC/HRMS (mzXML, mzML, netCDF) data where
  redundant isotope peaks inflate peak lists and only carbon-containing compound signals
  (12C monoisotopic isotopologues) should be retained.
when_to_use_negative:
- Input is already a peak-aligned feature table or peaklist from a prior peak-detection
  run — isotopologue pairing operates on raw full-scan MS1 spectra, not on post-detection
  tables.
- Data were acquired in MS/MS (MS2) fragmentation mode without MS1 full-scan spectral
  information — the approach relies on MS1-level 12C/13C isotope clusters in individual
  scans.
- The analysis targets non-carbon-containing inorganic species or pure isotope-label-free
  flux experiments where carbon isotope-pairing signatures are absent or intentionally
  removed.
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: IDSL.IPA
  role: Primary R package implementing the 12C/13C isotopologue ion-pairing algorithm
    (IPA_workflow) and downstream peak detection on EICs
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: R
  role: Runtime environment in which the IDSL.IPA package and IPA_workflow function
    are executed
- name: IDSL.UFA
  role: Downstream molecular formula annotation that consumes IDSL.IPA peaklists produced
    after isotopologue pairing
  repo: https://github.com/idslme/IDSL.UFA
- name: IDSL.UFAx
  role: Exhaustive chemical enumeration-based annotation of peaks from IDSL.IPA peaklists
  repo: https://github.com/idslme/IDSL.UFAx
- name: IDSL.CSA
  role: Downstream deconvolution of fragmentation spectra that can integrate with
    IDSL.IPA workflow outputs
  repo: https://github.com/idslme/IDSL.CSA
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:b50c284ec03b08c4b4a9d00ed3dcb99370d0e6063aa013dc1b622e8fde2b5b89
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.CSA
  - IDSL.IPA
  - IDSL.UFA
  - IDSL.UFAx
  - MS-DIAL
  - R
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# isotopologue-ion-pairing

## Summary

Pairs 12C and 13C isotopologues within individual mass spectra of untargeted LC/HRMS data to generate candidate monoisotopic m/z values, removing redundant isotope peaks before chromatographic peak detection.

## When to use

Use when processing untargeted LC/HRMS (mzXML, mzML, netCDF) data where redundant isotope peaks inflate peak lists and only carbon-containing compound signals (12C monoisotopic isotopologues) should be retained.

## When NOT to use

- Input is already a peak-aligned feature table or peaklist from a prior peak-detection run — isotopologue pairing operates on raw full-scan MS1 spectra, not on post-detection tables.
- Data were acquired in MS/MS (MS2) fragmentation mode without MS1 full-scan spectral information — the approach relies on MS1-level 12C/13C isotope clusters in individual scans.
- The analysis targets non-carbon-containing inorganic species or pure isotope-label-free flux experiments where carbon isotope-pairing signatures are absent or intentionally removed.

## Inputs

- MS1-level LC/HRMS raw data files (mzXML, mzML, or netCDF format) from untargeted metabolomics/exposomics studies
- IPA parameter spreadsheet (IPA_parameters.xlsx) with PARAM0007 set to the HRMS data location and PARAM0010 to the output location
- Number of processing threads (PARAM0006) for parallel peak detection and alignment

## Outputs

- Individual peaklist table (e.g. peaklist_003.csv) containing detected peaks with m/z, retention time, and chromatographic peak quality metrics
- Aligned peak intensity matrix across multiple samples for population-scale studies
- Candidate monoisotopic m/z values from 12C/13C isotopologue pairing in each scan

## How to apply

Run the IPA_workflow function from the IDSL.IPA R package, which first pairs all 12C/13C isotopologues in each individual mass spectrum to generate a list of candidate m/z values and removes redundant isotope peaks. Only the m/z values corresponding to the 12C isotopologue are used for the peak detection step on extracted ion chromatograms (EICs). Each individual 12C m/z peak is assigned to only one m/z cluster and is not reused in overlapping m/z clusters, preventing double-counting. This isotopologue-pairing strategy is applied prior to peak quality filtering (S/N, peak width, asymmetry thresholds), so peaks that fail the IDSL.IPA isotopologue pairing and peak quality thresholds are discarded before downstream peak alignment.

## Related tools

- **IDSL.IPA** (Primary R package implementing the 12C/13C isotopologue ion-pairing algorithm (IPA_workflow) and downstream peak detection on EICs) — https://cran.r-project.org/package=IDSL.IPA
- **R** (Runtime environment in which the IDSL.IPA package and IPA_workflow function are executed)
- **IDSL.UFA** (Downstream molecular formula annotation that consumes IDSL.IPA peaklists produced after isotopologue pairing) — https://github.com/idslme/IDSL.UFA
- **IDSL.UFAx** (Exhaustive chemical enumeration-based annotation of peaks from IDSL.IPA peaklists) — https://github.com/idslme/IDSL.UFAx
- **IDSL.CSA** (Downstream deconvolution of fragmentation spectra that can integrate with IDSL.IPA workflow outputs) — https://github.com/idslme/IDSL.CSA

## Examples

```
install.packages("IDSL.IPA"); library(IDSL.IPA); IPA_workflow("Address of the IPA parameter spreadsheet")
```

## Evaluation signals

- Total peak count should drop substantially relative to unfiltered peak lists because redundant isotope peaks are removed — e.g. IDSL.IPA detected 20,253 total peaks for a single test LC/HRMS file (003.d) from MTBLS1684, whereas 23,740 peaks were detected before recursive mass correction.
- Peaks in the output peaklist should all correspond to 12C monoisotopic isotopologue m/z values, not to 13C satellite peaks.
- Each 12C m/z peak should appear in exactly one m/z cluster in the generated candidate m/z list; verify no m/z is reused across overlapping clusters.
- Peaks that failed isotopologue pairing or peak quality thresholds (S/N, peak width, asymmetry) should be absent from the final peaklist.
- Running IPA_workflow on the same input file with identical parameters should reproduce the same peak count and peaklist, confirming deterministic isotopologue pairing.

## Limitations

- A majority of the rejected EICs had low abundance but passed the S/N ratio threshold; additional rules are needed to automatically filter out these rejected low abundant peaks.
- IDSL.IPA aligned data matrices require further processing for identifying adducts and fragments, filtering for statistics and annotating molecular formulas by additional computational tools.
- The isotopologue pairing approach removes redundant isotope peaks by detecting only the most abundant monoisotopic isotopologues, which may exclude non-carbon-containing species that lack a 12C/13C pairing signature.

## Evidence

- [results] Low-abundance peaks passing S/N still rejected: [evidence span withheld — anchor status: not_found]
