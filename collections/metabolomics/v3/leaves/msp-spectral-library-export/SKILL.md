---
name: msp-spectral-library-export
description: Use when you have IDSL.IPA peaklists and peak_alignment directories from
  MS1-only mzXML/mzML/netCDF data and need to group correlating EICs into CSA clusters
  and write them as a reusable .msp spectral library.
when_to_use_negative:
- Input is already a .msp/.mgf spectra library or feature table — use IDSL.FSA for
  spectral similarity searching instead of re-deconvolving
- You need genuine MS2 fragmentation spectra; CSA clusters are composite MS1 spectra
  built from correlating ions, not MS/MS scans
- Peak detection has not been performed — run the IDSL.IPA workflow first; IDSL.CSA
  requires peaklists and peak_alignment directories as input
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3318
tools:
- name: IDSL.CSA
  role: Performs the CSA correlation-based deconvolution of co-eluting MS1 ions and
    exports the clusters as .msp composite spectra
  repo: https://cran.r-project.org/package=IDSL.CSA
- name: IDSL.IPA
  role: Upstream peak detection that generates the peaklists and peak_alignment directories
    consumed by the CSA workflow
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.FSA
  role: Downstream mass spectral similarity search of the exported .msp CSA library
    against MS/MS reference libraries
  repo: https://cran.r-project.org/package=IDSL.FSA
- name: R
  role: Runtime environment; IDSL.CSA is an R CRAN package invoked via IDSL.CSA_workflow()
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:4ada40aa620c3815a6e362e18451946834e80bf3bb15d2ed0f8df6a7836962cf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# msp-spectral-library-export

## Summary

Exports IDSL.CSA correlation-based deconvolution clusters of co-eluting MS1 ions from LC/HRMS data into a .msp composite mass spectra library that can be searched with IDSL.FSA against MS/MS libraries such as NIST 2020.

## When to use

Use when you have IDSL.IPA peaklists and peak_alignment directories from MS1-only mzXML/mzML/netCDF data and need to group correlating EICs into CSA clusters and write them as a reusable .msp spectral library.

## When NOT to use

- Input is already a .msp/.mgf spectra library or feature table — use IDSL.FSA for spectral similarity searching instead of re-deconvolving
- You need genuine MS2 fragmentation spectra; CSA clusters are composite MS1 spectra built from correlating ions, not MS/MS scans
- Peak detection has not been performed — run the IDSL.IPA workflow first; IDSL.CSA requires peaklists and peak_alignment directories as input

## Inputs

- MS1-level HRMS data files (mzXML, mzML, netCDF), e.g. pool_P1_A1-A12.mzML from MSV000088661
- IDSL.IPA 'peaklists' directory (per-file chromatographic peak information, m/z-RT)
- IDSL.IPA 'peak_alignment' directory (aligned peaklists)
- IDSL.CSA parameter spreadsheet (CSA_parameters.xlsx) with paths filled in

## Outputs

- Composite spectra library as a .msp file in the CSA_MSP directory (e.g., 42 CSA spectra for the 12-standard test file)
- CSA_adduct_annotation: peaklists with potential adduct information
- peak_alignment_subset: subsets of aligned peak tables for major ions in each CSA cluster
- aligned_spectra_table: information for the CSA aggregation on the aligned peak table
- Batch aligned EIC figures for the CSA analysis

## How to apply

First run IDSL.IPA on the raw MS1 data to generate the 'peaklists' and 'peak_alignment' directories, then download the IDSL.CSA parameter spreadsheet (39 parameters in 5 sections) and set PARAM0001=YES for CSA-only processing, CSA0005 to the HRMS data location, CSA0008 to the peaklists directory, CSA0009 to the peak_alignment directory, CSA0011 to the output location, and optionally CSA0004 for thread count. Run the IDSL.CSA_workflow on the spreadsheet in R; the deconvolution groups correlated EICs (e.g., Pearson correlation coefficient threshold of 0.98, which grouped 76.9% of peaks in the pool_P1_A1-A12.mzML 12-standard test file into 42 EIC groups). Parse results at the CSA0011 address: the CSA_MSP subdirectory holds the .msp file of composite spectra, with companion outputs CSA_adduct_annotation, peak_alignment_subset, and aligned_spectra_table for verifying the aggregation on the aligned peak table.

## Related tools

- **IDSL.CSA** (Performs the CSA correlation-based deconvolution of co-eluting MS1 ions and exports the clusters as .msp composite spectra) — https://cran.r-project.org/package=IDSL.CSA
- **IDSL.IPA** (Upstream peak detection that generates the peaklists and peak_alignment directories consumed by the CSA workflow) — https://github.com/idslme/IDSL.IPA
- **IDSL.FSA** (Downstream mass spectral similarity search of the exported .msp CSA library against MS/MS reference libraries) — https://cran.r-project.org/package=IDSL.FSA
- **R** (Runtime environment; IDSL.CSA is an R CRAN package invoked via IDSL.CSA_workflow())

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("Address of the CSA parameter spreadsheet")
```

## Evaluation signals

- The .msp file exists in the CSA_MSP subdirectory at the CSA0011 output address and parses as valid MSP format
- Expected cluster counts are reproduced on benchmarks: the pool_P1_A1-A12.mzML test file yields 42 CSA spectra, with 76.9% of detected peaks grouped at Pearson correlation 0.98
- Downstream NIST 2020 search via IDSL.FSA reproduces the reported top hits for the 5 high-confidence clusters (Purine, L-Kynurenine, N-Acetyl histidine, Homoserine, N-Acetylneuraminic acid)
- aligned_spectra_table is consistent with CSA aggregation on the aligned peak table; peak_alignment_subset contains major-ion subsets for each cluster
- CSA_adduct_annotation peaklists contain plausible adduct flags consistent with the aligned m/z values

## Limitations

- Deconvolution outcome is threshold-sensitive: at Pearson correlation 0.98 only 76.9% of detected peaks in the 12-standard test file were grouped, so some peaks remain unclustered
- CSA composite spectra are derived from MS1-only correlated ions, so annotation quality depends on correlation structure rather than true fragment spectra
- No changelog was found for the software, so version-specific behavior changes are not documented

## Evidence

- [results] Applying a Pearson correlation coefficient threshold of 0.98 between EICs grouped 76.9% of the detected peaks into 42 EIC groups (CSA clusters), which were exported as 42 CSA spectra in MSP format.: [evidence span withheld — anchor status: not_found]
