---
name: chromatogram-deconvolution
description: Use when you have MS1-only LC/HRMS data (mzXML, mzML, netCDF) from untargeted
  metabolomics and need to deconvolve co-eluting MS1 ions into composite spectra without
  DDA MS2 files.
when_to_use_negative:
- You already have DDA MS2 spectra and need true tandem fragmentation spectra rather
  than MS1 composite spectra
- Input is already a peak/feature table produced by another peak picker (e.g. MZmine,
  xcms, MS-DIAL) with no IDSL.IPA peaklists/peak_alignment directories
- The goal is searching spectra against reference libraries (that is IDSL.FSA's role,
  downstream of CSA library creation)
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IDSL.CSA
  role: Core R package performing peak detection, chromatogram deconvolution, and
    composite spectra aggregation/export to .msp
  repo: https://github.com/idslme/IDSL.CSA
- name: IDSL.IPA
  role: 'Upstream peak detection: extracts chromatographic peaks (m/z-RT) and generates
    peaklists and peak_alignment directories required as CSA input'
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.FSA
  role: Downstream companion package to annotate the resulting .msp/.mgf files via
    mass spectral entropy, cosine similarity, and NEME criteria
  repo: https://github.com/idslme/IDSL.FSA
- name: R
  role: Runtime environment in which the IDSL.CSA workflow is invoked via library(IDSL.CSA)
  repo: https://cran.r-project.org/package=IDSL.CSA
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:267564e612044712b4acef60b0e9a583934a0603bade2a43b8afa5c396dca640
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.CSA
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# chromatogram-deconvolution

## Summary

Group co-eluting, correlating MS1 extracted ion chromatograms into composite spectra (CSA clusters) using the IDSL.CSA R package, so that LC/HRMS peaks lacking MS2 fragmentation can still be exported as re-usable .msp spectral libraries for chemical annotation.

## When to use

Use when you have MS1-only LC/HRMS data (mzXML, mzML, netCDF) from untargeted metabolomics and need to deconvolve co-eluting MS1 ions into composite spectra without DDA MS2 files.

## When NOT to use

- You already have DDA MS2 spectra and need true tandem fragmentation spectra rather than MS1 composite spectra
- Input is already a peak/feature table produced by another peak picker (e.g. MZmine, xcms, MS-DIAL) with no IDSL.IPA peaklists/peak_alignment directories
- The goal is searching spectra against reference libraries (that is IDSL.FSA's role, downstream of CSA library creation)

## Inputs

- Raw MS1-level LC/HRMS data files (mzXML, mzML, netCDF)
- IDSL.IPA 'peaklists' directory (per-file chromatographic peak information, m/z-RT)
- IDSL.IPA 'peak_alignment' directory (aligned peak table)
- IDSL.CSA parameter spreadsheet (CSA_parameters.xlsx, 39 parameters in 5 sections)

## Outputs

- CSA_MSP directory containing a .msp composite spectra library
- CSA_adduct_annotation: peaklists with potential adduct information
- peak_alignment_subset: subsets of aligned peak tables for major ions in each CSA cluster
- aligned_spectra_table: information for the CSA aggregation on the aligned table
- Batch aligned extracted ion chromatogram (EIC) figures

## How to apply

First run IDSL.IPA on the raw MS1 files to generate per-file 'peaklists' and aligned 'peak_alignment' directories carrying m/z-RT chromatographic information. Then download the IDSL.CSA parameter spreadsheet (39 parameters in 5 sections), set PARAM0001 = YES for CSA-only processing, point CSA0005 to the HRMS data, CSA0008 to the peaklists directory, CSA0009 to the peak_alignment directory, CSA0011 to the output location, and optionally CSA0004 to set processing threads. Run IDSL.CSA_workflow on the spreadsheet; the workflow groups peaks whose EICs correlate — in the MSV000088661 pooled-standards case a Pearson correlation coefficient threshold of 0.98 grouped 76.9% of detected peaks into 42 EIC clusters — and exports each cluster as a composite spectrum. The workflow also supports DDA and DIA (MSE, AIF, SWATH-MS) deconvolution modes and parallel processing on Windows and Linux.

## Related tools

- **IDSL.CSA** (Core R package performing peak detection, chromatogram deconvolution, and composite spectra aggregation/export to .msp) — https://github.com/idslme/IDSL.CSA
- **IDSL.IPA** (Upstream peak detection: extracts chromatographic peaks (m/z-RT) and generates peaklists and peak_alignment directories required as CSA input) — https://github.com/idslme/IDSL.IPA
- **IDSL.FSA** (Downstream companion package to annotate the resulting .msp/.mgf files via mass spectral entropy, cosine similarity, and NEME criteria) — https://github.com/idslme/IDSL.FSA
- **R** (Runtime environment in which the IDSL.CSA workflow is invoked via library(IDSL.CSA)) — https://cran.r-project.org/package=IDSL.CSA

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("Address of the CSA parameter spreadsheet")
```

## Evaluation signals

- The CSA0011 output address contains a CSA_MSP directory with a non-empty .msp file
- On the MSV000088661 pooled-standards test file (pool_P1_A1-A12.mzML), a Pearson correlation threshold of 0.98 yields 42 EIC groups/42 CSA spectra covering ~76.9% of detected peaks
- On the 54-file MSV000088661 standards study, 214 of 359 standards (59.6%) receive a CSA spectrum and up to 42 (19.6%) contain in-source fragments similar to MS/MS reference spectra
- Companion outputs (CSA_adduct_annotation, peak_alignment_subset, aligned_spectra_table) are present and consistent with aggregation on the aligned peak table

## Limitations

- Not every standard yields a composite spectrum: only 214 of 359 authentic standards (59.6%) were associated with a CSA spectrum, and only up to 42 of those (19.6%) had in-source fragments resembling MS/MS reference spectra
- CSA spectra capture in-source fragmentation at MS1 level, which may not match MS/MS libraries for all compounds
- Data must first be processed by IDSL.IPA; CSA cannot start directly from raw files or arbitrary third-party peak tables

## Evidence

- [results] Applying a Pearson correlation coefficient threshold of 0.98 between EICs grouped 76.9% of the detected peaks into 42 EIC groups (CSA clusters), which were exported as 42 CSA spectra in MSP format.: [evidence span withheld — anchor status: not_found]
- [abstract] generates composite mass spectra libraries from MS1-only data, enabling the chemical annotation of LC/HRMS peaks regardless of the availability of MS2 fragmentation spectra: "generates composite mass spectra libraries from MS1-only data, enabling the chemical annotation of LC/HRMS peaks regardless of the availability of MS2 fragmentation spectra"
- [readme] mass spectrometry data should be processed using the [IDSL.IPA] workflow to acquire chromatographic information of the peaks (m/z-RT): [evidence span withheld — anchor status: not_found]
- [results] 214 of the 359 standards (59.6%) were associated with a CSA spectrum by the workflow, and only up to 42 (19.6%) of those CSA spectra contained in-source fragments similar to MS/MS spectra: [evidence span withheld — anchor status: not_found]
