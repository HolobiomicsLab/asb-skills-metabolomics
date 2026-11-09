---
name: in-source-fragment-matching-to-msms
description: Use when annotating LC/HRMS peaks from MS1-only or DDA/DIA untargeted
  metabolomics data where MS2 fragmentation spectra are absent or sparse, and you
  want to leverage in-source fragments within composite spectra as surrogates for
  MS/MS library matches.
when_to_use_negative:
- Every detected peak of interest already has high-quality DDA MS/2 fragmentation
  spectra that can be searched directly against MS/MS libraries without composite
  spectra
- Input is already a peak/feature table without the underlying m/z-RT chromatographic
  information required by IDSL.IPA/IDSL.CSA (no raw mzXML/mzML/netCDF or peaklists
  to aggregate)
- The goal is de novo compound identification rather than library-based similarity
  annotation of peaks
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: IDSL.CSA
  role: R package that performs peak detection, chromatogram deconvolution, and composite
    spectra aggregation from MS1-only data to build the CSA .msp library
  repo: https://github.com/idslme/IDSL.CSA
- name: IDSL.IPA
  role: Upstream R package that extracts peaks (m/z-RT chromatographic information,
    peaklists, aligned peak table) from raw LC/HRMS files as input to CSA aggregation
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.FSA
  role: Companion R package that searches the exported .msp CSA library against MS/MS
    reference libraries using mass spectral entropy similarity, dot product (cosine)
    similarity, and NEME
  repo: https://cran.r-project.org/package=IDSL.FSA
- name: R
  role: Runtime environment in which the IDSL.CSA/IPA/FSA workflows are executed
  repo: https://cran.r-project.org/package=IDSL.CSA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:3f4aa66243e13c3f4158bab6dbc5f000d6182db218f3249685c40f40ff9c016e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.CSA
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# in-source-fragment-matching-to-msms

## Summary

Generates composite spectra (CSA) libraries from MS1-only LC/HRMS data by grouping correlating co-eluting ions (isotopologues, adducts, in-source fragments), then matches those in-source fragments against MS/MS reference libraries (e.g. NIST 2020) via IDSL.FSA mass spectral similarity searches to annotate peaks lacking MS2 data.

## When to use

Use when annotating LC/HRMS peaks from MS1-only or DDA/DIA untargeted metabolomics data where MS2 fragmentation spectra are absent or sparse, and you want to leverage in-source fragments within composite spectra as surrogates for MS/MS library matches.

## When NOT to use

- Every detected peak of interest already has high-quality DDA MS/2 fragmentation spectra that can be searched directly against MS/MS libraries without composite spectra
- Input is already a peak/feature table without the underlying m/z-RT chromatographic information required by IDSL.IPA/IDSL.CSA (no raw mzXML/mzML/netCDF or peaklists to aggregate)
- The goal is de novo compound identification rather than library-based similarity annotation of peaks

## Inputs

- Raw LC/HRMS untargeted metabolomics files (mzXML, mzML, netCDF) in ESI mode
- IDSL.IPA peaklists directory (per-file peaklists with m/z-RT chromatographic information)
- IDSL.IPA peak_alignment directory (aligned peak table)
- IDSL.CSA parameter spreadsheet (CSA_parameters.xlsx)
- MS/MS reference mass spectral library (e.g. NIST 2020) in .msp/.mgf format for similarity search

## Outputs

- CSA_MSP directory containing the composite spectra library as a .msp file
- CSA_adduct_annotation: peaklists with potential adduct information
- peak_alignment_subset: subsets of the aligned peak table for major ions in each CSA cluster
- aligned_spectra_table: information for CSA aggregation on the aligned table
- Annotated peaks: counts of standards/peaks whose in-source fragments match MS/MS reference spectra

## How to apply

First process raw mzXML/mzML/netCDF files with the IDSL.IPA workflow to obtain per-file peaklists and an aligned peak table carrying m/z-RT chromatographic information. Download the IDSL.CSA parameter spreadsheet, set PARAM0001 = YES (CSA-only workflow), and point CSA0005 to the HRMS MS1 data location, CSA0008 to the peaklists directory, CSA0009 to the peak_alignment directory, CSA0011 to the output location, and CSA0004 to available processing threads. Run IDSL.CSA_workflow to perform peak detection, chromatogram deconvolution, and composite spectra aggregation in ESI mode, exporting a CSA_MSP (.msp) library, CSA_adduct_annotation, peak_alignment_subset, and aligned_spectra_table. Then search the .msp CSA library against MS/MS reference libraries (e.g. NIST 2020) using IDSL.FSA, which scores matches by mass spectral entropy similarity, dot product (cosine) similarity, and normalized Euclidean mass error (NEME). Judge success by whether in-source fragments within the composite spectra resemble MS/MS reference spectra; in the reference benchmark, 214/359 authentic standards (59.6%) yielded a CSA spectrum and up to 42 (19.6%) of those contained in-source fragments similar to MS/MS library spectra.

## Related tools

- **IDSL.CSA** (R package that performs peak detection, chromatogram deconvolution, and composite spectra aggregation from MS1-only data to build the CSA .msp library) — https://github.com/idslme/IDSL.CSA
- **IDSL.IPA** (Upstream R package that extracts peaks (m/z-RT chromatographic information, peaklists, aligned peak table) from raw LC/HRMS files as input to CSA aggregation) — https://github.com/idslme/IDSL.IPA
- **IDSL.FSA** (Companion R package that searches the exported .msp CSA library against MS/MS reference libraries using mass spectral entropy similarity, dot product (cosine) similarity, and NEME) — https://cran.r-project.org/package=IDSL.FSA
- **R** (Runtime environment in which the IDSL.CSA/IPA/FSA workflows are executed) — https://cran.r-project.org/package=IDSL.CSA

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("CSA_parameters.xlsx")
```

## Evaluation signals

- Validation from the benchmark run: the CSA_MSP directory contains a .msp library, and reproduced counts match the reported 214/359 standards (59.6%) with CSA spectra and 42 (19.6%) standards whose in-source fragments resemble MS/MS reference spectra
- All four expected output artifacts exist at the CSA0011 output location: CSA_MSP (.msp), CSA_adduct_annotation, peak_alignment_subset, and aligned_spectra_table
- CSA adduct annotation correctly assigns precursor types/molecular formulas consistent with the 359 authentic standards' known precursor m/z and reverse-phase retention time
- Similarity search results are reproducible using IDSL.FSA's three matching criteria (mass spectral entropy similarity, dot product/cosine similarity, NEME) against NIST 2020
- Annotation rates for commonly detected endogenous metabolites in human blood samples are comparable between IDSL.CSA libraries and MS/MS libraries in validation tests

## Limitations

- Not every standard produces a usable composite spectrum: in the reference benchmark only 59.6% (214/359) of standards were associated with a CSA spectrum, and only up to 19.6% (42) of those CSA spectra contained in-source fragments similar to MS/MS reference library spectra
- Co-eluting ions at MS1 level can be exported as spectra by related grouping approaches (RAMClust, eISA, CorrDec, IIMN, MS-FLO, MetaboAnatoR), so CSA composite spectra may include spurious co-eluting fragments
- Existing MS2 libraries poorly cover the expected chemical space, limiting the reference side of the in-source fragment matching

## Evidence

- [methods] IDSL.IPA preprocessing is required before CSA: [evidence span withheld — anchor status: not_found]
- [readme] FSA matching metrics: [evidence span withheld — anchor status: external_doc]
