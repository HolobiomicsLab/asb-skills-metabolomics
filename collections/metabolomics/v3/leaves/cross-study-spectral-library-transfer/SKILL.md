---
name: cross-study-spectral-library-transfer
description: Use when an .msp CSA library built from one study's MS1-only data is
  available and the goal is to annotate peaks in an independent untargeted LC/HRMS
  study beyond the compounds already recovered by MS/MS spectral library search.
when_to_use_negative:
- The query study has no composite spectra (.msp) yet and the task is to build the
  CSA library from raw LC/HRMS files — use the IDSL.IPA + IDSL.CSA library-creation
  workflow first
- The goal is de novo identification with no reference spectral library from a related
  study available
- You need confirmatory compound identification; CSA matches are MS1-derived composite
  spectra similarities, not MS2-based confirmations
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.CSA
  role: Creates the composite spectra (CSA) .msp libraries from MS1-only LC/HRMS data
    and exports the unique spectra variants searched across studies
  repo: https://github.com/idslme/IDSL.CSA
- name: IDSL.FSA
  role: Performs the mass spectral similarity search of query .msp composite spectra
    against the reference CSA .msp library using entropy/cosine/NEME criteria
  repo: https://github.com/idslme/IDSL.FSA
- name: IDSL.IPA
  role: Preprocesses raw mzXML/mzML/netCDF files to generate the peaklists and peak
    alignment needed before CSA spectra can be created
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Runtime environment in which the IDSL.CSA and IDSL.FSA parameter-spreadsheet-driven
    workflows are executed
  repo: https://cran.r-project.org/package=IDSL.CSA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:d86dec12342b41ea345939d60d452e94879b22bf79f523b465bf210648f3680a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.CSA
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cross-study-spectral-library-transfer

## Summary

Search a composite spectra (CSA) .msp library generated from one untargeted LC/HRMS metabolomics study against CSA spectra variants exported from a different study using IDSL.FSA, recovering cross-study compound annotations that MS/MS library searches (e.g. NIST 2020) miss.

## When to use

Use when an .msp CSA library built from one study's MS1-only data is available and the goal is to annotate peaks in an independent untargeted LC/HRMS study beyond the compounds already recovered by MS/MS spectral library search.

## When NOT to use

- The query study has no composite spectra (.msp) yet and the task is to build the CSA library from raw LC/HRMS files — use the IDSL.IPA + IDSL.CSA library-creation workflow first
- The goal is de novo identification with no reference spectral library from a related study available
- You need confirmatory compound identification; CSA matches are MS1-derived composite spectra similarities, not MS2-based confirmations

## Inputs

- Unique CSA spectra variants (.msp) exported for the query study (e.g. ST001000)
- CSA composite spectra library (.msp) previously generated for the reference study (e.g. ST000923)
- Existing MS/MS library search annotations for the query study (e.g. NIST 2020 hits)
- Aligned peak table with chromatographic information (m/z-RT) from IDSL.IPA

## Outputs

- CSV table of additional annotated compounds (unique InChIKeys) not covered by the MS/MS library search
- Annotations aggregated on the aligned peak table with meta-variables (InChIKey, SMILES, precursor type, molecular formula)

## How to apply

First export the unique CSA spectra variants (.msp) for the query study — these are produced by running IDSL.IPA on the raw mzXML/mzML/netCDF files followed by the IDSL.CSA workflow with its parameter spreadsheet (e.g. set PARAM0001 to YES for CSA-only mode and point CSA0005/0008/0009/0011 at the HRMS data, peaklists, peak_alignment, and output directories). Load the reference CSA .msp library previously generated for the second study. Configure the IDSL.FSA parameter spreadsheet and run the FSA workflow in R to search the query composite spectra against the reference library; FSA matches .msp/.mgf entries using mass spectral entropy similarity, dot product (cosine) similarity, and normalized Euclidean mass error (NEME) with intelligent pre-filtering, optionally using retention time and adduct type. Aggregate the matched annotations on the aligned peak table using meta-variables such as InChIKey, SMILES, precursor type, and molecular formula as carried in the reference library. Finally, subtract compounds already annotated by the query study's MS/MS library search (e.g. NIST 2020 hits) to isolate the newly annotated compounds, and write the result to a CSV table.

## Related tools

- **IDSL.CSA** (Creates the composite spectra (CSA) .msp libraries from MS1-only LC/HRMS data and exports the unique spectra variants searched across studies) — https://github.com/idslme/IDSL.CSA
- **IDSL.FSA** (Performs the mass spectral similarity search of query .msp composite spectra against the reference CSA .msp library using entropy/cosine/NEME criteria) — https://github.com/idslme/IDSL.FSA
- **IDSL.IPA** (Preprocesses raw mzXML/mzML/netCDF files to generate the peaklists and peak alignment needed before CSA spectra can be created) — https://github.com/idslme/IDSL.IPA
- **R** (Runtime environment in which the IDSL.CSA and IDSL.FSA parameter-spreadsheet-driven workflows are executed) — https://cran.r-project.org/package=IDSL.CSA

## Examples

```
library(IDSL.FSA); IDSL.FSA_workflow("Address of the FSA parameter spreadsheet")  # spreadsheet points at ST001000 unique CSA .msp as query and ST000923 CSA library .msp as reference
```

## Evaluation signals

- The output CSV contains the expected count of unique additional InChIKeys after subtraction — for the ST001000-vs-ST000923 case study this is exactly 34 additional InChIKeys
- Every reported annotation carries the reference library meta-variables (InChIKey, SMILES, precursor type, molecular formula)
- No InChIKey in the final additional-annotation list overlaps the compounds already recovered by the query study's MS/MS library search (e.g. NIST 2020)
- Matches are scored by FSA's matching variables (mass spectral entropy similarity, dot product cosine similarity, NEME) rather than unmatched entries
- The searched .msp files contain only the unique CSA spectra variants for the query study, not redundant duplicates

## Limitations

- Transferred annotations depend on the chemical space covered by the reference study's CSA library; available spectral libraries only partially cover the expected chemical space
- CSA spectra are MS1-derived composites, so cross-study transfer works best when studies share acquisition modes and chemistry; composite-similarity matches are not MS2-confirmed identifications
- Not all LC/HRMS peaks can be annotated — peaks lacking a matching library entry remain unannotated

## Evidence

- [readme] FSA matching criteria: [evidence span withheld — anchor status: external_doc]
