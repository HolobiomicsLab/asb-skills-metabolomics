---
name: mass-spectral-library-generation
description: Use when you have MS1-level LC/HRMS mzXML/mzML/netCDF data from an untargeted
  metabolomics study (possibly n>500) and need to build a reusable spectral library
  for annotating peaks, especially when MS2 data or MS2 library coverage is poor.
when_to_use_negative:
- You already have rich MS2 fragmentation spectra for the peaks and a matching MS/MS
  spectral library (e.g. NIST 2020) — searching MS/MS libraries directly is more appropriate
  than building CSA libraries.
- The input is raw unprocessed MS data without IDSL.IPA peaklists/peak_alignment outputs
  — CSA requires IDSL.IPA-generated chromatographic information as a prerequisite.
- You need compound-level identification solely from accurate mass and formula without
  spectral similarity search — CSA libraries are designed for mass spectral similarity
  matching, not de novo annotation.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: IDSL.CSA
  role: Core R package that performs peak detection, chromatogram deconvolution, and
    composite spectra (CSA) aggregation from IDSL.IPA peaklists, exporting the .msp
    library and adduct-annotation tables.
  repo: https://cran.r-project.org/package=IDSL.CSA
- name: IDSL.IPA
  role: Upstream R package that processes mzXML/mzML/netCDF raw data to produce the
    peaklists and peak_alignment directories consumed by the CSA workflow.
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.FSA
  role: Companion R package used to search the generated .msp CSA library (mass spectral
    entropy similarity, dot product cosine similarity, NEME) for chemical annotation
    of peaks.
  repo: https://cran.r-project.org/package=IDSL.FSA
- name: R
  role: Runtime environment in which IDSL.CSA_workflow is executed from the CRAN-installed
    package.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:6cdae2c4e18bd638a4d9161bac52f5bf75c46435a34f8d9e29895bb6b15c4f13
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mass-spectral-library-generation

## Summary

Generate composite mass spectra (CSA) libraries in .msp format from correlating MS1-only LC/HRMS peaks using the IDSL.CSA R package, enabling chemical annotation of peaks that lack MS2 fragmentation data.

## When to use

Use when you have MS1-level LC/HRMS mzXML/mzML/netCDF data from an untargeted metabolomics study (possibly n>500) and need to build a reusable spectral library for annotating peaks, especially when MS2 data or MS2 library coverage is poor.

## When NOT to use

- You already have rich MS2 fragmentation spectra for the peaks and a matching MS/MS spectral library (e.g. NIST 2020) — searching MS/MS libraries directly is more appropriate than building CSA libraries.
- The input is raw unprocessed MS data without IDSL.IPA peaklists/peak_alignment outputs — CSA requires IDSL.IPA-generated chromatographic information as a prerequisite.
- You need compound-level identification solely from accurate mass and formula without spectral similarity search — CSA libraries are designed for mass spectral similarity matching, not de novo annotation.

## Inputs

- MS1-level LC/HRMS raw mass spectrometry data in mzXML, mzML, or netCDF format
- IDSL.IPA-generated 'peaklists' directory (chromatographic information of individual peaks, m/z-RT)
- IDSL.IPA-generated 'peak_alignment' directory (aligned peaklists)
- IDSL.CSA parameter spreadsheet (CSA_parameters.xlsx) with the CSA section filled in (CSA0005, CSA0008, CSA0009, CSA0011, CSA0004)
- Optional reference library for aggregating annotated chemical structures via InChIKey, SMILES, precursor type, molecular formula (PARAM0006)

## Outputs

- Composite spectra library as a .msp file in the CSA_MSP output folder
- CSA_adduct_annotation peaklists with potential adduct information
- peak_alignment_subset: subsets of aligned peak tables for the major ions in each CSA cluster
- aligned_spectra_table: CSA aggregation information on the aligned table
- Batch untargeted aligned extracted ion chromatogram (EIC) figures

## How to apply

First process raw mass spectrometry data (mzXML, mzML, netCDF) with the IDSL.IPA workflow to acquire chromatographic information of the peaks (m/z-RT), producing a 'peaklists' directory and a 'peak_alignment' directory. Install IDSL.CSA from CRAN (install.packages('IDSL.CSA')), download the CSA parameter spreadsheet, and configure the CSA section: set PARAM0001 to YES to run only the CSA workflow, point CSA0005 to the MS1-level HRMS data location, CSA0008 to the IDSL.IPA 'peaklists' directory, CSA0009 to the IDSL.IPA 'peak_alignment' directory, CSA0011 to the output location, and optionally raise the number of processing threads via CSA0004. Then run library(IDSL.CSA); IDSL.CSA_workflow('<address of the CSA parameter spreadsheet>'), which performs peak detection and chromatogram deconvolution to aggregate correlating MS1 ions into composite spectra and exports the library as a .msp file in the CSA_MSP folder. The rationale is that correlating ions in MS1-only data (isotopologues, adducts, in-source fragments) can be grouped into composite spectra that behave like searchable mass spectra, providing chemical annotation where MS2 fragmentation spectra are unavailable.

## Related tools

- **IDSL.CSA** (Core R package that performs peak detection, chromatogram deconvolution, and composite spectra (CSA) aggregation from IDSL.IPA peaklists, exporting the .msp library and adduct-annotation tables.) — https://cran.r-project.org/package=IDSL.CSA
- **IDSL.IPA** (Upstream R package that processes mzXML/mzML/netCDF raw data to produce the peaklists and peak_alignment directories consumed by the CSA workflow.) — https://github.com/idslme/IDSL.IPA
- **IDSL.FSA** (Companion R package used to search the generated .msp CSA library (mass spectral entropy similarity, dot product cosine similarity, NEME) for chemical annotation of peaks.) — https://cran.r-project.org/package=IDSL.FSA
- **R** (Runtime environment in which IDSL.CSA_workflow is executed from the CRAN-installed package.)

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow('CSA_parameters.xlsx')
```

## Evaluation signals

- The output directory CSA0011 must contain a CSA_MSP folder with a parseable .msp composite spectra library file.
- The CSA_adduct_annotation folder contains peaklists with potential adduct information consistent with the input aligned peak table.
- peak_alignment_subset tables exist for the major ions in each CSA cluster, and aligned_spectra_table records the CSA aggregation on the aligned table.
- On the ST001000 study (n=226), full-scale CSA yielded 139,018 total CSA spectra with a median of 631 CSA spectra per file, and ~31% of MS1 peaklist peaks were part of CSA spectra — comparable counts per file scale and % peaks covered indicate correct parameterization.
- Generated composite spectra libraries should show comparable annotation rates for commonly detected endogenous metabolites in human blood samples versus MS/MS libraries in validation tests.

## Limitations

- CSA deconvolution and library construction depend on IDSL.IPA's peak detection quality; poor peak picking (e.g. low sensitivity/specificity in complex matrices) will propagate to the composite spectra.
- CSA libraries built from MS1-only data may not perfectly match MS/MS library annotations, since composite spectra are surrogate representations; validation showed comparable but not necessarily identical annotation rates.
- Cross-applicability across independent studies depends on the study design and instrument conditions; libraries created from one LC/HRMS platform may not transfer perfectly to another.
- Large population-scale studies (n>500) require attention to CSA0004 thread count and computational resources for parallel processing in Windows and Linux.

## Evidence
