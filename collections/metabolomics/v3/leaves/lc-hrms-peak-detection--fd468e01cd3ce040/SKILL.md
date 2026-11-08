---
name: lc-hrms-peak-detection
description: Use when raw LC/HRMS files (mzXML, mzML, netCDF) must be converted into
  per-file peaklists and an aligned peak table carrying m/z-RT chromatographic information
  before MS1-level deconvolution (e.g., IDSL.CSA) or annotation.
when_to_use_negative:
- Input is already a feature/peak table or aligned peaklist — IDSL.IPA operates on
  raw mzXML/mzML/netCDF files
- Data are MS/MS (MS2) acquisition files needing MS2 spectra extraction rather than
  MS1-level peak detection and chromatographic information
- Peak picking is needed for gas chromatography MS with EI spectra not covered by
  the LC/HRMS EIC-based workflow assumptions
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IDSL.IPA
  role: 'Primary peak-detection engine: extracts chromatographic peaks (m/z-RT) from
    raw LC/HRMS files, producing the peaklists and peak_alignment directories consumed
    by IDSL.CSA'
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.CSA
  role: Downstream consumer of IDSL.IPA peaklists and aligned peak table for composite
    spectra deconvolution
  repo: https://github.com/idslme/IDSL.CSA
- name: R
  role: Runtime environment in which IDSL.IPA (and the CSA workflow) is executed
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:c3d0b3ccc15af95f6f2e83b50f4068d5d66a12e903cedc518892979bc0c350c6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lc-hrms-peak-detection

## Summary

Detect chromatographic peaks and build peaklists/aligned peak tables (m/z-RT) from untargeted LC/HRMS data (mzXML, mzML, netCDF) using the IDSL.IPA R package, as the mandatory precursor step for downstream IDSL.CSA composite spectra deconvolution.

## When to use

Use when raw LC/HRMS files (mzXML, mzML, netCDF) must be converted into per-file peaklists and an aligned peak table carrying m/z-RT chromatographic information before MS1-level deconvolution (e.g., IDSL.CSA) or annotation.

## When NOT to use

- Input is already a feature/peak table or aligned peaklist — IDSL.IPA operates on raw mzXML/mzML/netCDF files
- Data are MS/MS (MS2) acquisition files needing MS2 spectra extraction rather than MS1-level peak detection and chromatographic information
- Peak picking is needed for gas chromatography MS with EI spectra not covered by the LC/HRMS EIC-based workflow assumptions

## Inputs

- Raw untargeted LC/HRMS data files in mzXML, mzML, or netCDF format
- IDSL.IPA parameter spreadsheet (IPA_parameters.xlsx)
- For targeted use: reference annotations with confirmed retention time and precursor m/z values

## Outputs

- 'peaklists' directory with per-file chromatographic peak lists (m/z-RT)
- 'peak_alignment' directory with the aligned peak table (m/z-RT) used downstream by IDSL.CSA

## How to apply

Process raw mass spectrometry data (mzXML, mzML, netCDF) with the IDSL.IPA R package (installed via install.packages('IDSL.IPA'), parameters set through the IPA parameter spreadsheet) to generate 'peaklists' and 'peak_alignment' directories containing chromatographic m/z-RT information for individual and aggregated aligned peaklists. IDSL.IPA covers EIC candidate generation, peak detection, peak property evaluation, recursive mass correction, retention time correction across multiple batches, and peak annotation, and has been reported to outperform MZmine 2, xcms, and MS-DIAL in sensitivity, specificity, and speed. The resulting peaklists directory feeds CSA0008 and the peak_alignment directory feeds CSA0009 of the IDSL.CSA parameter spreadsheet, so outputs must be kept in the directory structure CSA expects. This step applies to both MS1-only studies (e.g., the 600-file ST000923 stool study) and pooled-standard runs (e.g., pool_P1_A1-A12.mzML from MSV000088661).

## Related tools

- **IDSL.IPA** (Primary peak-detection engine: extracts chromatographic peaks (m/z-RT) from raw LC/HRMS files, producing the peaklists and peak_alignment directories consumed by IDSL.CSA) — https://github.com/idslme/IDSL.IPA
- **IDSL.CSA** (Downstream consumer of IDSL.IPA peaklists and aligned peak table for composite spectra deconvolution) — https://github.com/idslme/IDSL.CSA
- **R** (Runtime environment in which IDSL.IPA (and the CSA workflow) is executed)

## Examples

```
library(IDSL.IPA); IPA_workflow("Address of the IPA parameter spreadsheet")
```

## Evaluation signals

- The 'peaklists' and 'peak_alignment' directories exist and contain per-file peaklists plus an aligned peak table with m/z and RT columns
- Downstream sanity: on the MSV000088661 pooled-standards test file (pool_P1_A1-A12.mzML), IDSL.IPA peak detection followed by CSA deconvolution at Pearson correlation 0.98 yields 42 EIC groups / 42 CSA spectra
- On population-scale runs (e.g., 600 ST000923 stool LC/HRMS files), the aligned table supports the published scale of deconvolution (25,351 CSA spectra) without processing failures
- Retention time correction across batches has been applied when data span multiple batches (an explicit IDSL.IPA feature)

## Limitations

- IDSL.IPA output must be regenerated whenever the CSA run needs consistent peaklists/peak_alignment directories; CSA parameters CSA0008/CSA0009 point directly at these directories
- Peak lists are MS1-level only — they carry no fragmentation information, so subsequent annotation relies on correlation-based composite spectra rather than true MS/MS
- IDSL.IPA's sensitivity/specificity claims were benchmarked against MZmine 2, xcms, and MS-DIAL for LC/HRMS organic small molecules; performance outside that domain is not established here

## Evidence

- [readme] IDSL.IPA is a suite of new algorithms covering extracted ion chromatogram (EIC) candidate generation, peak detection, peak property evaluation, recursive mass correction, retention time correction: [evidence span withheld — anchor status: external_doc]
- [readme] IDSL.IPA is able to outperform similar peak picking tools such as MZmine 2, *xcms*, and MS-DIAL in terms of sensitivity, specificity and speed: [evidence span withheld — anchor status: external_doc]
