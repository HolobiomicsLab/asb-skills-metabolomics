---
name: inchikey-annotation-aggregation
description: Use when you have matched annotations from a spectra search (e.g. CSA
  spectra searched against an .msp library) on an aligned peak table and need to consolidate
  them per compound (InChIKey) and identify annotations beyond those recovered by
  MS/MS library searches.
when_to_use_negative:
- The study data already have comprehensive MS2 spectra and the goal is only standard
  MS/MS spectral library searching rather than extending annotations from MS1-level
  composite spectra
- The input is raw mass spectrometry data (mzXML/mzML/netCDF) rather than searched
  spectra and an aligned peak table — peak detection (IDSL.IPA) and library creation/searching
  must precede this aggregation step
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: IDSL.FSA
  role: Performs the mass spectral similarity search of the query .msp spectra against
    the reference CSA .msp library and annotates various types of MSP files
  repo: https://cran.r-project.org/package=IDSL.FSA
- name: IDSL.CSA
  role: Generates the composite spectra (CSA) libraries and exports the .msp spectra
    variants that are searched and aggregated
  repo: https://cran.r-project.org/package=IDSL.CSA
- name: R
  role: Execution environment for the IDSL.CSA/IDSL.FSA workflows and the InChIKey-level
    aggregation/subtraction of annotations
  repo: https://cran.r-project.org/package=IDSL.CSA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:a0a2978090068ebece0be273cf9ffcdcfc5f653f0b132b1891f91fefefa7715d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - R
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# inchikey-annotation-aggregation

## Summary

Aggregates annotated chemical structures on an aligned LC/HRMS peak table using meta-variables (InChIKey, SMILES, precursor type, molecular formula) after IDSL.FSA similarity searches, so cross-study CSA library hits can be compared with and subtracted from existing MS/MS library annotations to isolate newly annotated compounds.

## When to use

Use when you have matched annotations from a spectra search (e.g. CSA spectra searched against an .msp library) on an aligned peak table and need to consolidate them per compound (InChIKey) and identify annotations beyond those recovered by MS/MS library searches.

## When NOT to use

- The study data already have comprehensive MS2 spectra and the goal is only standard MS/MS spectral library searching rather than extending annotations from MS1-level composite spectra
- The input is raw mass spectrometry data (mzXML/mzML/netCDF) rather than searched spectra and an aligned peak table — peak detection (IDSL.IPA) and library creation/searching must precede this aggregation step

## Inputs

- Unique CSA spectra variants (.msp) exported for the query study (e.g. ST001000)
- Reference CSA library (.msp) generated from a prior study (e.g. ST000923)
- Aligned peak table from the query study (IDSL.IPA/CSA peak_alignment output)
- Existing MS/MS spectral library search results (e.g. NIST 2020 hits) for the query study
- IDSL.FSA parameter spreadsheet (FSA_parameters.xlsx)

## Outputs

- Matched annotations aggregated on the aligned peak table with InChIKey, SMILES, precursor type, and molecular formula meta-variables
- CSV table of additional unique InChIKeys annotated by the CSA library but not by the MS/MS library search

## How to apply

Run the IDSL.FSA mass spectral similarity search (combining mass spectral entropy similarity, dot product/cosine similarity, and normalized Euclidean mass error criteria) of the query spectra variants (.msp) against the reference CSA library (.msp). Then aggregate the matched annotations on the aligned peak table using meta-variables such as InChIKey, SMILES, precursor type, and molecular formula, as supported by the IDSL.CSA/FSA pipeline. Finally, compare the CSA-library-derived InChIKeys with hits from an existing MS/MS spectral library (e.g. NIST 2020) for the same study and subtract previously annotated compounds, writing the remaining unique InChIKeys to a CSV table. The rationale is that composite spectra from MS1-only correlating peaks can annotate compounds lacking MS2 data, and the InChIKey level is the correct granularity to deduplicate adduct/isomer variants into unique chemical structures.

## Related tools

- **IDSL.FSA** (Performs the mass spectral similarity search of the query .msp spectra against the reference CSA .msp library and annotates various types of MSP files) — https://cran.r-project.org/package=IDSL.FSA
- **IDSL.CSA** (Generates the composite spectra (CSA) libraries and exports the .msp spectra variants that are searched and aggregated) — https://cran.r-project.org/package=IDSL.CSA
- **R** (Execution environment for the IDSL.CSA/IDSL.FSA workflows and the InChIKey-level aggregation/subtraction of annotations) — https://cran.r-project.org/package=IDSL.CSA

## Examples

```
library(IDSL.FSA); IDSL.FSA_workflow("FSA_parameter_spreadsheet.xlsx")  # search ST001000 unique CSA spectra .msp against ST000923 CSA library .msp, then aggregate hits by InChIKey on the aligned peak table
```

## Evaluation signals

- The output CSV contains the expected number of unique additional InChIKeys (e.g. exactly 34 for ST001000 searched against the ST000923 CSA library, matching the reported result)
- Every reported additional InChIKey is absent from the study's existing MS/MS library (e.g. NIST 2020) hit list, confirming the subtraction was performed correctly
- Aggregated rows carry consistent InChIKey, SMILES, precursor type, and molecular formula meta-variables sourced from the reference library
- Matched annotations are successfully joined to the aligned peak table (each hit maps to an aligned peak entry)

## Limitations

- Not all LC/HRMS peaks can be annotated because MS2 fragmentation data are not available for every detected peak and available MS2 libraries poorly cover the expected chemical space
- CSA-library matches provide annotation-level evidence (not confirmation), and cross-study applicability depends on the compounds and acquisition conditions being comparable between studies
- Existing MS1 annotation approaches are limited to flagging peaks as isotopologues, adducts, or in-source fragments, so CSA matches should be interpreted with adduct/fragment ambiguity in mind

## Evidence

- [readme] IDSL.FSA similarity metrics: [evidence span withheld — anchor status: external_doc]
