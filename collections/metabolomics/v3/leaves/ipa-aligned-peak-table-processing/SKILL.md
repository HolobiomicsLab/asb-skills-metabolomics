---
name: ipa-aligned-peak-table-processing
description: Use when raw untargeted LC/HRMS files (mzXML, mzML, netCDF) must be converted
  into IDSL.IPA-generated peaklists and peak_alignment tables before running IDSL.CSA
  deconvolution.
when_to_use_negative:
- When the input is already an IDSL.IPA-aligned peak table and only the CSA deconvolution
  step itself needs to be run — re-running IPA peak detection is unnecessary.
- When MS/MS (DDA) spectra-based annotation via IDSL.FSA is the goal and MS1-only
  composite spectra are not needed.
- When raw data is not LC/HRMS (e.g. direct infusion MS or GC-only data without chromatographic
  RT information).
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.IPA
  role: Upstream peak-detection engine that generates peaklists and peak_alignment
    directories from raw mzXML/mzML/netCDF data; its outputs are the required input
    for IDSL.CSA.
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.CSA
  role: Downstream consumer of the IPA peaklists and peak_alignment directories; driven
    by a parameter spreadsheet via IDSL.CSA_workflow().
  repo: https://github.com/idslme/IDSL.CSA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:f4ac8aa32ef22de2221de67482fa675cf7d6c26c6c236b59a485ad55f1c582ab
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.CSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ipa-aligned-peak-table-processing

## Summary

Acquires chromatographic peak information (m/z-RT) from raw mzXML/mzML/netCDF LC/HRMS data with IDSL.IPA and consumes the resulting peaklists and peak_alignment directories as the upstream prerequisite for IDSL.CSA composite spectra deconvolution.

## When to use

Use when raw untargeted LC/HRMS files (mzXML, mzML, netCDF) must be converted into IDSL.IPA-generated peaklists and peak_alignment tables before running IDSL.CSA deconvolution.

## When NOT to use

- When the input is already an IDSL.IPA-aligned peak table and only the CSA deconvolution step itself needs to be run — re-running IPA peak detection is unnecessary.
- When MS/MS (DDA) spectra-based annotation via IDSL.FSA is the goal and MS1-only composite spectra are not needed.
- When raw data is not LC/HRMS (e.g. direct infusion MS or GC-only data without chromatographic RT information).

## Inputs

- Raw LC/HRMS data files (mzXML, mzML, netCDF) at MS1 level
- IDSL.IPA-generated peaklists directory
- IDSL.IPA-generated peak_alignment directory
- IDSL.CSA parameter spreadsheet

## Outputs

- Individual and aggregated aligned peak tables (peaklists) with m/z-RT chromatographic information
- Aligned_spectra_table.csv (CSA aggregation results)
- CSA_MSP directory containing .msp files
- peak_alignment_subset tables for major ions in each CSA cluster

## How to apply

First process the raw mass spectrometry data with the IDSL.IPA workflow to extract peaks for organic small molecules, obtaining chromatographic information of the peaks (m/z-RT) as individual and aggregated aligned peaklists plus peak_alignment directories. Install IDSL.CSA from CRAN, download the IDSL.CSA parameter spreadsheet, and configure the CSA-specific path parameters: CSA0008 for the IDSL.IPA peaklists directory, CSA0009 for the peak_alignment directory, CSA0005 for the MS1-level HRMS data location, and CSA0011 for the output location, with PARAM0001 set to YES to run only the CSA workflow. The number of processing threads can be increased via CSA0004. The rationale is that IDSL.CSA operates on the IDSL.IPA peak tables rather than raw spectra, so accurate peak picking, recursive mass correction, and RT correction across batches by IDSL.IPA directly condition the downstream CSA seed-peak selection and EIC correlation steps.

## Related tools

- **IDSL.IPA** (Upstream peak-detection engine that generates peaklists and peak_alignment directories from raw mzXML/mzML/netCDF data; its outputs are the required input for IDSL.CSA.) — https://github.com/idslme/IDSL.IPA
- **IDSL.CSA** (Downstream consumer of the IPA peaklists and peak_alignment directories; driven by a parameter spreadsheet via IDSL.CSA_workflow().) — https://github.com/idslme/IDSL.CSA

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("Address of the CSA parameter spreadsheet")
```

## Evaluation signals

- Downstream CSA workflow runs without missing-input errors, confirming the peaklists and peak_alignment paths (CSA0008/CSA0009) point to valid IDSL.IPA outputs.
- CSA output directories exist as expected: CSA_MSP (.msp), CSA_adduct_annotation, peak_alignment_subset, and aligned_spectra_table.csv.
- aligned_spectra_table.csv contains the expected CSA aggregation columns with non-trivial cluster counts.
- Parameter spreadsheet values match the README quick-batch defaults (PARAM0001=YES, CSA0004 for threads, CSA0005/0008/0009/0011 paths).
- On the ST002263 quick-batch example (n = 33, Thermo Q Exactive HF, HILIC-ESI-POS/NEG), the pipeline completes for both polarities.

## Limitations

- IDSL.IPA and IDSL.CSA are designed for population-scale untargeted studies (n > 500); smaller datasets may require adjusted parameter defaults.
- Raw data must be one of mzXML, mzML, or netCDF; other vendor formats are not supported by the workflow as described.
- The CSA workflow requires the 39-parameter spreadsheet to be fully configured; incorrect path parameters (e.g. CSA0008/CSA0009) will break the pipeline.
- License information for the IDSL.IPA repo is not documented in the provided context.

## Evidence

- [readme] CSA0008 for Address of the peaklists directory generated by the IDSL.IPA workflow; CSA0009 for Address of the peak_alignment directory.: [evidence span withheld — anchor status: not_found]
- [readme] IDSL.IPA generates comprehensive and high-quality datasets from untargeted analysis of organic small molecules for population-size studies.: [evidence span withheld — anchor status: external_doc]
- [methods] mass spectrometry data should be processed using the [IDSL.IPA](https://github.com/idslme/IDSL.IPA) workflow to acquire chromatographic information: [evidence span withheld — anchor status: not_found]
- [readme] IDSL.IPA is able to outperform similar peak picking tools such as MZmine 2, xcms, and MS-DIAL in terms of sensitivity, specificity and speed.: [evidence span withheld — anchor status: not_found]
