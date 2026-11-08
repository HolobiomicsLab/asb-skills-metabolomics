---
name: composite-spectra-deconvolution-from-ms1-peaks
description: Use when input is MS1-only LC/HRMS data (mzXML/mzML/netCDF) with IDSL.IPA
  peaklists and peak_alignment output, and the goal is composite-spectra (.msp) generation
  or annotation without MS2 fragmentation.
when_to_use_negative:
- Input data contain usable MS2/DDA or DIA (MSE, AIF, SWATH-MS) spectra — use the
  corresponding DDA or DIA analysis modes instead, or a standard MS/MS library search
- You only need peak detection itself; run IDSL.IPA directly rather than the CSA deconvolution
  step
- The input is already a composite spectra (.msp) library — use IDSL.FSA for spectral
  similarity search and annotation, not CSA.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: IDSL.CSA
  role: R package that performs peak detection and chromatogram deconvolution to deconvolute
    correlating MS1 ions into composite spectra and export .msp libraries, adduct
    annotations, aligned-spectra tables and EICs
  repo: https://github.com/idslme/IDSL.CSA
- name: IDSL.IPA
  role: Upstream step that extracts peaks (m/z–RT) and generates individual/aggregated
    peaklists and the peak_alignment directory that CSA consumes
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.FSA
  role: Downstream companion for searching the resulting .msp libraries and generating
    fragmentation libraries
  repo: https://github.com/idslme/IDSL.FSA
- name: R
  role: Runtime environment for the IDSL.CSA package workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:a804e58139dc6761f636e4dfa39f441a59ef0d0bcb50faa0f50c2d12deebdfcf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.CSA
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# composite-spectra-deconvolution-from-ms1-peaks

## Summary

Deconvolves MS1-only LC/HRMS peaklists into composite spectra (CSA) by aggregating correlating co-eluting ions per chromatographic peak, producing reusable .msp libraries for chemical annotation when MS2 data are absent. Implemented in the IDSL.CSA R package after IDSL.IPA peak detection.

## When to use

Use when input is MS1-only LC/HRMS data (mzXML/mzML/netCDF) with IDSL.IPA peaklists and peak_alignment output, and the goal is composite-spectra (.msp) generation or annotation without MS2 fragmentation.

## When NOT to use

- Input data contain usable MS2/DDA or DIA (MSE, AIF, SWATH-MS) spectra — use the corresponding DDA or DIA analysis modes instead, or a standard MS/MS library search
- You only need peak detection itself; run IDSL.IPA directly rather than the CSA deconvolution step
- The input is already a composite spectra (.msp) library — use IDSL.FSA for spectral similarity search and annotation, not CSA.

## Inputs

- MS1-only LC/HRMS raw data (mzXML, mzML, netCDF)
- IDSL.IPA individual and aggregated peaklists (peaklists directory)
- IDSL.IPA peak_alignment directory
- IDSL.CSA parameter spreadsheet (CSA_parameters.xlsx)

## Outputs

- Composite spectra library (.msp files) in CSA_MSP
- Peaklists with potential adduct information (CSA_adduct_annotation)
- Aligned peak-table subsets for major ions in each CSA cluster (peak_alignment_subset)
- CSA aggregation information on the aligned table (aligned_spectra_table)
- Batch untargeted aligned extracted ion chromatogram (EIC) figures

## How to apply

First process raw MS1 data with IDSL.IPA to obtain individual peaklists, aggregated peaklists, and the peak_alignment directory. Then download the CSA_parameters.xlsx spreadsheet, set PARAM0001=Yes to run the CSA workflow only, and fill CSA0004 (threads), CSA0005 (MS1 HRMS data location), CSA0008 (IPA peaklists dir), CSA0009 (IPA peak_alignment dir), and CSA0011 (output dir), keeping default chromatogram-deconvolution parameters as validated on ST001000 (n=226, HILIC-ESI-POS, Thermo Q-Exactive Orbitrap) and ST002263 (n=33, Q Exactive HF). Run IDSL.CSA_workflow on the spreadsheet; it performs peak detection and chromatogram deconvolution to group correlating MS1 ions into composite spectra, then aggregates adduct and structure meta-variables (InChIKey, SMILES, precursor type, molecular formula) on the aligned peak table. The method is justified because it enables chemical annotation of LC/HRMS peaks regardless of MS2 availability, achieving annotation rates comparable to MS/MS libraries for common human blood endogenous metabolites.

## Related tools

- **IDSL.CSA** (R package that performs peak detection and chromatogram deconvolution to deconvolute correlating MS1 ions into composite spectra and export .msp libraries, adduct annotations, aligned-spectra tables and EICs) — https://github.com/idslme/IDSL.CSA
- **IDSL.IPA** (Upstream step that extracts peaks (m/z–RT) and generates individual/aggregated peaklists and the peak_alignment directory that CSA consumes) — https://github.com/idslme/IDSL.IPA
- **IDSL.FSA** (Downstream companion for searching the resulting .msp libraries and generating fragmentation libraries) — https://github.com/idslme/IDSL.FSA
- **R** (Runtime environment for the IDSL.CSA package workflow)

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("Address of the CSA parameter spreadsheet")
```

## Evaluation signals

- CSA_MSP folder contains a non-empty .msp composite spectra library
- ST001000 full-scale run reproduces 139,018 total CSA spectra with median 631 per file; 16,659 unique CSA spectra with ≥1% detection frequency
- On average ~31% of MS1 peaklist peaks fall within CSA spectra across all samples
- aligned_spectra_table and peak_alignment_subset are populated and consistent with the aligned peak table
- Downstream adduct/InChIKey/name annotation counts on the aligned table are comparable to those obtained with MS/MS libraries

## Limitations

- Not every LC/HRMS peak becomes an annotated composite spectrum; only ~31% of MS1 peaklist peaks were part of CSA spectra in the ST001000 benchmark
- Annotation quality depends on the reference library content (InChIKey/SMILES/adduct meta-variables), and MS2 libraries only poorly cover the expected chemical space
- Requires IDSL.IPA pre-processing; raw files cannot be fed directly to IDSL.CSA
- Parameter selection (39 parameters over 5 sections) must be tuned; the paper's validation covers HILIC-ESI-POS Thermo Q-Exactive Orbitrap human blood studies

## Evidence

- [results] ST001000 benchmark: [evidence span withheld — anchor status: not_found]
