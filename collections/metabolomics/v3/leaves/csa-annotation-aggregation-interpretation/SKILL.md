---
name: csa-annotation-aggregation-interpretation
description: Use when an untargeted LC/HRMS study lacks MS2 fragmentation or MS2-library
  coverage, and you need to annotate peaks by mass spectral similarity of composite
  spectra and then aggregate/interpret the annotation at the chemical-class level.
when_to_use_negative:
- The dataset already has comprehensive MS2 fragmentation spectra and a matching MS2
  spectral library covering the expected chemical space, so IDSL.CSA composite spectra
  adds little value.
- The input is a single chromatogram or non-LC/HRMS acquisition (e.g. direct infusion)
  that does not produce correlating MS1 peaks for composite spectra aggregation.
- The user needs quantitation of known targets rather than untargeted chemical annotation
  of LC/HRMS peaks.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: IDSL.CSA
  role: Core package that creates composite spectra libraries from MS1-only data and
    aggregates annotated chemical structures on the aligned peak table via meta-variables
    such as InChIKey, SMILES, precursor type and molecular formula
  repo: https://cran.r-project.org/package=IDSL.CSA
- name: IDSL.IPA
  role: Pre-processing step that extracts chromatographic peak information (m/z-RT)
    from raw mzXML/mzML/netCDF data to feed the IDSL.CSA workflow
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.FSA
  role: Searches the generated CSA .msp/.mgf composite spectra libraries against reference
    libraries using mass spectral entropy, dot-product (cosine) similarity and NEME
    criteria
  repo: https://github.com/idslme/IDSL.FSA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:83104381d6ee29a4634a026e8a34cdd4001c71ff9633e9e35a9178c053495cbd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ChemRICH
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# csa-annotation-aggregation-interpretation

## Summary

End-to-end IDSL.CSA pipeline for annotating untargeted LC/HRMS peaks from MS1-only composite spectra and interpreting the resulting annotations by aggregating chemical structures (via InChIKey/SMILES/formula) into class-level biological insight.

## When to use

Use when an untargeted LC/HRMS study lacks MS2 fragmentation or MS2-library coverage, and you need to annotate peaks by mass spectral similarity of composite spectra and then aggregate/interpret the annotation at the chemical-class level.

## When NOT to use

- The dataset already has comprehensive MS2 fragmentation spectra and a matching MS2 spectral library covering the expected chemical space, so IDSL.CSA composite spectra adds little value.
- The input is a single chromatogram or non-LC/HRMS acquisition (e.g. direct infusion) that does not produce correlating MS1 peaks for composite spectra aggregation.
- The user needs quantitation of known targets rather than untargeted chemical annotation of LC/HRMS peaks.

## Inputs

- Raw LC/HRMS data files in mzXML, mzML or netCDF format (MS1 level)
- IDSL.IPA-generated peaklists directory and peak_alignment directory
- IDSL.CSA parameter spreadsheet (CSA_parameters.xlsx)
- Reference spectral library files in .msp or .mgf format for IDSL.FSA annotation
- Aligned peak table from an untargeted metabolomics study

## Outputs

- CSA_MSP directory containing .msp composite spectra library files
- CSA_adduct_annotation peaklists with potential adduct information
- peak_alignment_subset tables for major ions in each CSA cluster
- aligned_spectra_table with CSA aggregation results on the aligned table
- Aggregated chemical-structure annotation keyed by InChIKey, SMILES, precursor type and molecular formula

## How to apply

Process raw mzXML/mzML/netCDF data with IDSL.IPA to obtain chromatographic peak information (m/z-RT) for individual and aligned peaklists. Download the IDSL.CSA parameter spreadsheet, set CSA0005 (MS1 data location), CSA0008 (peaklists directory), CSA0009 (peak_alignment directory), CSA0011 (output), and set PARAM0001 to YES for the CSA workflow; run IDSL.CSA_workflow on that spreadsheet. CSA aggregates correlating MS1 peaks into composite mass spectra libraries, which IDSL.FSA searches against .msp/.mgf reference libraries using mass spectral entropy similarity, dot-product (cosine) similarity and normalized Euclidean mass error (NEME). The unique CSA feature aggregates annotated chemical structures on the aligned peak table using meta-variables such as InChIKey, SMILES, precursor type and molecular formula, enabling class-level interpretation of the annotation results.

## Related tools

- **IDSL.CSA** (Core package that creates composite spectra libraries from MS1-only data and aggregates annotated chemical structures on the aligned peak table via meta-variables such as InChIKey, SMILES, precursor type and molecular formula) — https://cran.r-project.org/package=IDSL.CSA
- **IDSL.IPA** (Pre-processing step that extracts chromatographic peak information (m/z-RT) from raw mzXML/mzML/netCDF data to feed the IDSL.CSA workflow) — https://github.com/idslme/IDSL.IPA
- **IDSL.FSA** (Searches the generated CSA .msp/.mgf composite spectra libraries against reference libraries using mass spectral entropy, dot-product (cosine) similarity and NEME criteria) — https://github.com/idslme/IDSL.FSA

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("Address of the CSA parameter spreadsheet")
```

## Evaluation signals

- CSA_MSP output directory must contain a valid .msp file with composite spectra entries.
- CSA_adduct_annotation output must contain peaklists with potential adduct information for each peak.
- aligned_spectra_table must be populated and show aggregation of the aligned peak table into CSA clusters.
- Annotation rates for commonly detected endogenous metabolites in human blood should be comparable to those obtained using MS/MS libraries in validation tests.
- CSA workflow completes on population-size studies (n > 500) and generates batch untargeted aligned EIC figures.

## Limitations

- Annotation is limited to peaks that can be grouped into correlating MS1 ions; peaks with no MS1 neighbors may remain unannotated.
- Requires IDSL.IPA pre-processing; IDSL.CSA cannot ingest raw vendor files directly.
- Composite spectra libraries are generated from MS1-only data, so structural resolution is lower than true MS2 fragmentation spectra for distinguishing positional isomers.
- Performance claims and validation were demonstrated for human blood endogenous metabolites; other matrices may behave differently.

## Evidence

- [abstract] IDSL.CSA generates composite mass spectra libraries from MS1-only data, enabling chemical annotation of LC/HRMS peaks regardless of MS2 spectra availability: [evidence span withheld — anchor status: not_found]
- [readme] IDSL.FSA annotates .msp and .mgf files using a combination of mass spectral entropy similarity, dot product (cosine) similarity, and normalized Euclidean mass error (NEME) criteria: [evidence span withheld — anchor status: external_doc]
