---
name: ms1-peak-deconvolution
description: Use when processing LC/HRMS untargeted metabolomics datasets where peaks
  lack MS2 spectra or available MS2 libraries poorly cover the chemical space, and
  chemical annotation must be recovered from MS1-only data.
when_to_use_negative:
- The dataset already has comprehensive MS2 fragmentation spectra for all detected
  peaks and matches available MS2 libraries, making CSA deconvolution unnecessary.
- You need to process MS2-dependent (DDA) or MS2-independent (DIA) deconvolution rather
  than MS1-only composite spectra grouping - use IDSL.CSA's DDA or DIA modes instead
  of the CSA workflow.
- The input is raw vendor binary data that has not yet been peak-picked; IDSL.IPA
  processing must precede IDSL.CSA.
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IDSL.CSA
  role: R package implementing composite spectra deconvolution of MS1 peaks, generating
    .msp libraries and aligned_spectra_table.csv from IDSL.IPA peaklists
  repo: github.com/idslme/IDSL.CSA
- name: IDSL.IPA
  role: Upstream peak detection and deconvolution tool that produces the 'peaklists'
    and 'peak_alignment' directories consumed by the CSA workflow
  repo: github.com/idslme/IDSL.IPA
- name: IDSL.FSA
  role: Downstream companion package used to search the generated .msp composite spectra
    libraries against query spectra for chemical annotation
  repo: github.com/idslme/IDSL.FSA
- name: R
  role: Runtime environment in which IDSL.CSA_workflow is invoked
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:75f7e2a18b4ba4d7e46a0c860c0290a2a899ffa6a6a8d3198a66b84989cbb160
schema_version: 0.2.0
metadata:
  license_tier: unknown
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
yes_rate: 0.8235294117647058
n_claims: 17
---

# ms1-peak-deconvolution

## Summary

Deconvolves co-eluting MS1 chromatographic peaks from untargeted LC/HRMS data into composite spectra (CSA) clusters using IDSL.CSA, enabling chemical annotation even when no MS2 fragmentation data are available. The method relies on seed-peak selection, retention-time grouping, and Pearson correlation of smoothed EICs.

## When to use

Use when processing LC/HRMS untargeted metabolomics datasets where peaks lack MS2 spectra or available MS2 libraries poorly cover the chemical space, and chemical annotation must be recovered from MS1-only data.

## When NOT to use

- The dataset already has comprehensive MS2 fragmentation spectra for all detected peaks and matches available MS2 libraries, making CSA deconvolution unnecessary.
- You need to process MS2-dependent (DDA) or MS2-independent (DIA) deconvolution rather than MS1-only composite spectra grouping - use IDSL.CSA's DDA or DIA modes instead of the CSA workflow.
- The input is raw vendor binary data that has not yet been peak-picked; IDSL.IPA processing must precede IDSL.CSA.

## Inputs

- Raw LC/HRMS data in mzXML, mzML, or netCDF format
- IDSL.IPA 'peaklists' directory (per-sample chromatographic peak information)
- IDSL.IPA 'peak_alignment' directory (aligned m/z-RT peak table)
- IDSL.CSA parameter spreadsheet (CSA_parameters.xlsx) with 39 parameters in 5 sections

## Outputs

- CSA_MSP directory containing .msp composite spectra files
- CSA_adduct_annotation directory with peaklists annotated with potential adduct information
- peak_alignment_subset directory with subsets of aligned peak tables for major ions in each CSA cluster
- aligned_spectra_table.csv with information on CSA aggregation

## How to apply

First process raw mzXML, mzML, or netCDF data with IDSL.IPA to generate 'peaklists' and 'peak_alignment' directories containing m/z-RT chromatographic information. Then download the IDSL.CSA parameter spreadsheet (39 parameters across 5 sections, e.g. CSA0005 for HRMS data location, CSA0008/CSA0009 for peaklists/peak_alignment addresses, CSA0011 for output location), set PARAM0001 to YES for CSA-only processing, and run IDSL.CSA_workflow on the spreadsheet. For each seed peak ranked by signal-to-noise, gather co-eluting candidate neighbors within a retention-time window and compute Pearson correlations between LOESS-smoothed and cubic-spline interpolated EICs, retaining neighbors with correlation >= 0.98. Apply minimum ion count and m/z range filters, handle 12C/13C isotopologue pairs by expected m/z spacing, and aggregate qualifying neighbors into CSA clusters written to aligned_spectra_table.csv.

## Related tools

- **IDSL.CSA** (R package implementing composite spectra deconvolution of MS1 peaks, generating .msp libraries and aligned_spectra_table.csv from IDSL.IPA peaklists) — github.com/idslme/IDSL.CSA
- **IDSL.IPA** (Upstream peak detection and deconvolution tool that produces the 'peaklists' and 'peak_alignment' directories consumed by the CSA workflow) — github.com/idslme/IDSL.IPA
- **IDSL.FSA** (Downstream companion package used to search the generated .msp composite spectra libraries against query spectra for chemical annotation) — github.com/idslme/IDSL.FSA
- **R** (Runtime environment in which IDSL.CSA_workflow is invoked)

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("Address of the CSA parameter spreadsheet")
```

## Evaluation signals

- Output CSA_MSP directory contains a valid .msp file parseable by IDSL.FSA.
- aligned_spectra_table.csv exists and reports aggregation counts consistent with the reported validation numbers (e.g. 255 peaks, 153 neighbors, 42 clusters on the pooled-standards test file).
- CSA_adduct_annotation directory contains adduct-annotated peaklists; peak_alignment_subset contains per-cluster major-ion subsets.
- Each CSA cluster's member ions have pairwise Pearson correlation of the smoothed EICs >= 0.98 and fall within the configured RT window.
- 12C/13C isotopologue pairs in each cluster exhibit the expected m/z spacing rather than being treated as unrelated co-eluting ions.

## Limitations

- Requires prior IDSL.IPA processing; IDSL.CSA does not operate directly on raw vendor files.
- Existing MS1 annotation approaches (including this one) are limited to grouping correlating ions to flag peaks as potential isotopologues, adducts, or in-source fragments or assigning molecular formula.
- Not all LC/HRMS peaks are annotated because MS2 fragmentation data are not available for every detected peak and available MS2 libraries poorly cover the expected chemical space - CSA mitigates but does not eliminate annotation gaps.
- Only 42 clusters were recovered from 255 peaks in the pooled-standards test file, indicating that many co-eluting ion groups may fail the correlation or ion-count thresholds.

## Evidence
