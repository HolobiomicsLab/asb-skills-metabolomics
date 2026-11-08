---
name: targeted-compound-annotation-from-ms1-data
description: Use when you need to reproduce or reproduce-scale compound annotation
  from MS1-only LC/HRMS datasets (e.g., ST000923's 600 files / 177 annotations) by
  generating a CSA .msp spectra library in targeted mode and searching it with IDSL.FSA.
when_to_use_negative:
- The dataset already contains DDA MS2 fragmentation spectra that can be searched
  directly against conventional MS/MS libraries without composite MS1 deconvolution.
- You only have an aligned feature table and no raw mzXML/mzML/netCDF files or IDSL.IPA
  peaklists/peak_alignment directories to feed the CSA workflow.
- The goal is de novo formula assignment rather than annotation against a targeted
  list of known compounds or cross-study library search.
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: IDSL.CSA
  role: 'Core package: runs the Composite Spectra Analysis workflow in targeted mode
    from a 39-parameter spreadsheet, deconvoluting CSA spectra and writing the .msp
    library'
  repo: https://cran.r-project.org/package=IDSL.CSA
- name: IDSL.IPA
  role: 'Upstream preprocessing: peak detection and chromatogram deconvolution of
    mzXML/mzML/netCDF data to produce peaklists and peak_alignment directories'
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.FSA
  role: 'Downstream search: annotates the generated CSA .msp file against MS/MS mass
    spectral libraries using mass spectral entropy similarity, dot product similarity,
    and NEME criteria'
  repo: https://cran.r-project.org/package=IDSL.FSA
- name: R
  role: Runtime environment for installing and executing the IDSL.CSA_workflow function
  repo: https://cran.r-project.org/package=IDSL.CSA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:b5a1171f532f7b0d38373af2d3359983132eae7806d661176b0f25294f9d2a3c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# targeted-compound-annotation-from-ms1-data

## Summary

Applies the IDSL.CSA R package in targeted mode to build composite spectra (CSA) libraries from correlating MS1-only LC/HRMS ions against a reference compound list, then searches them via IDSL.FSA to annotate peaks without MS2 fragmentation data.

## When to use

Use when you need to reproduce or reproduce-scale compound annotation from MS1-only LC/HRMS datasets (e.g., ST000923's 600 files / 177 annotations) by generating a CSA .msp spectra library in targeted mode and searching it with IDSL.FSA.

## When NOT to use

- The dataset already contains DDA MS2 fragmentation spectra that can be searched directly against conventional MS/MS libraries without composite MS1 deconvolution.
- You only have an aligned feature table and no raw mzXML/mzML/netCDF files or IDSL.IPA peaklists/peak_alignment directories to feed the CSA workflow.
- The goal is de novo formula assignment rather than annotation against a targeted list of known compounds or cross-study library search.

## Inputs

- Raw MS1-level LC/HRMS data files (mzXML, mzML, netCDF)
- IDSL.IPA 'peaklists' directory (chromatographic m/z-RT peak information)
- IDSL.IPA 'peak_alignment' directory (aligned peaklists)
- IDSL.CSA parameter spreadsheet (CSA_parameters.xlsx, 39 parameters in 5 sections)
- Targeted reference library of published compound annotations (e.g., 177 annotations for ST000923 with InChIKey, SMILES, precursor type, molecular formula)

## Outputs

- CSA .msp composite spectra library (CSA_MSP folder)
- CSA_adduct_annotation peaklists with potential adduct information
- peak_alignment_subset tables (subsets of aligned peak tables for major ions in each CSA cluster)
- aligned_spectra_table (CSA aggregation on the aligned table)
- Batch aligned extracted ion chromatogram (EIC) figures
- IDSL.FSA search results: MS/MS library annotation hits for the CSA .msp

## How to apply

First process the raw mzXML/mzML/netCDF files with the IDSL.IPA workflow to generate the 'peaklists' and 'peak_alignment' directories containing chromatographic m/z-RT information. Download the IDSL.CSA parameter spreadsheet (39 parameters in 5 sections; CSA analysis requires ~39 parameters), set PARAM0001=YES for CSA-only processing, point CSA0005 to the HRMS data location, CSA0008/CSA0009 to the peaklists and peak_alignment directories, CSA0011 to the output location for .msp files and EICs, and CSA0004 to the number of processing threads. In targeted mode, supply the published annotations (e.g., 177 compounds for ST000923, with InChIKey/SMILES/precursor type/molecular formula meta-variables) as the reference library, then run IDSL.CSA_workflow() on the spreadsheet. Parse the CSA0011 output: CSA_MSP (.msp composite spectra library), CSA_adduct_annotation peaklists, peak_alignment_subset tables, and aligned_spectra_table. Finally, search the generated .msp against MS/MS mass spectral libraries (e.g., NIST 2020) with the IDSL.FSA workflow to count library hits.

## Related tools

- **IDSL.CSA** (Core package: runs the Composite Spectra Analysis workflow in targeted mode from a 39-parameter spreadsheet, deconvoluting CSA spectra and writing the .msp library) — https://cran.r-project.org/package=IDSL.CSA
- **IDSL.IPA** (Upstream preprocessing: peak detection and chromatogram deconvolution of mzXML/mzML/netCDF data to produce peaklists and peak_alignment directories) — https://github.com/idslme/IDSL.IPA
- **IDSL.FSA** (Downstream search: annotates the generated CSA .msp file against MS/MS mass spectral libraries using mass spectral entropy similarity, dot product similarity, and NEME criteria) — https://cran.r-project.org/package=IDSL.FSA
- **R** (Runtime environment for installing and executing the IDSL.CSA_workflow function) — https://cran.r-project.org/package=IDSL.CSA

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("Address of the CSA parameter spreadsheet")
```

## Evaluation signals

- Compound coverage matches the ST000923 benchmark: 105 of 177 targeted compounds (59.3%) yield at least one CSA spectrum.
- Total deconvoluted CSA spectra matches 25,351 from the 600 samples (median detection frequency 204, min 1, max 589).
- Mean CSA spectra variants per compound is approximately 2.5.
- IDSL.FSA similarity search of the generated .msp against MS/MS libraries (e.g., NIST 2020) returns 46 library hits.
- The CSA0011 output contains all expected artifacts: CSA_MSP (.msp), CSA_adduct_annotation, peak_alignment_subset, and aligned_spectra_table directories.

## Limitations

- Not all LC/HRMS peaks can be annotated because MS2 fragmentation data are not available for every detected peak, and available MS2 libraries poorly cover the expected chemical space.
- Benchmark metrics (105/177 compounds, 25,351 spectra, 46 FSA hits) are study-specific to ST000923's 600 files and default parameter values; other datasets will differ.
- Processing population-size studies (n > 500) is supported but runtime depends on the number of threads set via CSA0004 and available compute.
- Existing MS1-level approaches (RAMClust, eISA, CorrDec, IIMN, MS-FLO, MetaboAnnotatoR) only flag ions as isotopologues, adducts, or in-source fragments — CSA libraries are complementary but their cross-study applicability to new biological insights is still being established.

## Evidence

- [methods] download the [IDSL.CSA parameter spreadsheet] and select the parameters accordingly and then use this spreadsheet as the input for the IDSL.CSA workflow: [evidence span withheld — anchor status: not_found]
- [results] 105 of the 177 targeted compounds (59.3%) had an associated CSA spectrum, with a median detection frequency of 204 (min 1, max 589): [evidence span withheld — anchor status: not_found]
