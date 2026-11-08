---
name: spectral-library-matching
description: Use when Rapid QC-MS has produced MS-DIAL feature annotations from mzML
  files and you need to align features against a predefined .msp spectral library
  to quantify and annotate user-defined QC targets (RS or ES workflows).
when_to_use_negative:
- Input data are already annotated feature tables or QC metric CSVs — this skill operates
  on MS-DIAL feature annotations from raw/mzML data
- No .msp spectral library exists for the analytical matrix; matching requires a predefined
  library appropriate to the matrix type
- Data are not from Thermo acquisition — Rapid QC-MS and this matching pipeline have
  only been tested extensively on Thermo RAW files and sequences
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-DIAL ver.4.9.221218
  role: Performs spectral similarity scoring of feature m/z and MS/MS or EI spectra
    against the .msp library and produces quantification/annotation results
  repo: http://prime.psc.riken.jp/compms/msdial/main.html
- name: MSConvert 3.0.23332-7faa7f422
  role: Converts vendor Thermo .raw files into open mzML format before routing them
    to MS-DIAL for library matching
  repo: https://proteowizard.sourceforge.io/tools/msconvert.html
- name: Rapid QC-MS
  role: Orchestrates the job, hosts the .msp library configuration (Settings > Biological
    Standards), and visualizes the aligned annotation results on its dashboard
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:5e73303cea66f8502e6578a6a7e60be37dc7abe53e80e3eab2f8cac83eb38d92
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/czbiohub-sf/Rapid-QC-MS
  tools:
  - MS-DIAL ver.4.9.221218
  - MSConvert
  - MSConvert 3.0.23332-7faa7f422
  - Rapid QC-MS
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spectral-library-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Matches MS-DIAL feature annotations (m/z plus MS/MS or EI spectra) from converted mzML files against a user-supplied .msp spectral library, producing quantification/annotation results that feed Rapid QC-MS quality metrics and the RS/ES comparison modules.

## When to use

Use when Rapid QC-MS has produced MS-DIAL feature annotations from mzML files and you need to align features against a predefined .msp spectral library to quantify and annotate user-defined QC targets (RS or ES workflows).

## When NOT to use

- Input data are already annotated feature tables or QC metric CSVs — this skill operates on MS-DIAL feature annotations from raw/mzML data
- No .msp spectral library exists for the analytical matrix; matching requires a predefined library appropriate to the matrix type
- Data are not from Thermo acquisition — Rapid QC-MS and this matching pipeline have only been tested extensively on Thermo RAW files and sequences

## Inputs

- mzML files produced by MSConvert from completed Thermo .raw files
- Predefined .msp spectral library (analyte name, m/z, RT, MS2) matched to the reference-standard or external-standard analytical matrix
- MS-DIAL feature annotations exported by the Rapid QC-MS processing pipeline

## Outputs

- MS-DIAL target quantification/annotation results for all user-defined targets
- es_alignment_results.csv with match score, m/z error, and retention-time fields per ES feature
- rs_vs_msp_comparison.csv with one row per aligned RS-to-library feature match, including retention time shift and mass accuracy deviation

## How to apply

First convert completed Thermo .raw files to mzML with MSConvert (v3.0.23332-7faa7f422), since MS-DIAL (ver.4.9.221218) consumes the mzML output. Load the .msp library appropriate to the workflow: for the reference-standard (RS) workflow the library contains high-confidence annotations from previously analyzed reference specimens in a well-defined biological matrix, combined as analyte name, m/z, RT, and MS2 information; for the external-standard (ES) workflow use a library matched to the ES analytical matrix, kept as a distinct module from RS. Match each feature's m/z and MS/MS (or EI) spectrum against .msp entries using spectral similarity scoring in MS-DIAL, recording match score, m/z error, and retention-time agreement per feature. Write results as es_alignment_results.csv for the ES module or rs_vs_msp_comparison.csv for the RS module (one row per aligned RS-to-library feature match, including retention time shift and mass accuracy deviation). The rationale is that ES and RS are often not from the same biological matrix, so separate molecular-target libraries enable matrix-appropriate searches while preserving QC granularity.

## Related tools

- **MS-DIAL ver.4.9.221218** (Performs spectral similarity scoring of feature m/z and MS/MS or EI spectra against the .msp library and produces quantification/annotation results) — http://prime.psc.riken.jp/compms/msdial/main.html
- **MSConvert 3.0.23332-7faa7f422** (Converts vendor Thermo .raw files into open mzML format before routing them to MS-DIAL for library matching) — https://proteowizard.sourceforge.io/tools/msconvert.html
- **Rapid QC-MS** (Orchestrates the job, hosts the .msp library configuration (Settings > Biological Standards), and visualizes the aligned annotation results on its dashboard) — https://github.com/czbiohub-sf/Rapid-QC-MS

## Examples

```
rapidqcms  # configure a new QC job with the sequence .csv, library .msp, and Thermo .raw files; Rapid QC-MS then invokes MSConvert and MS-DIAL to match features against the library
```

## Evaluation signals

- es_alignment_results.csv exists and contains alignment fields (match score, m/z error, retention time) for queried ES features
- rs_vs_msp_comparison.csv exists with one row per aligned RS-to-library feature match, including retention time shift and mass accuracy deviation fields
- MS-DIAL produced quantified/annotated results for all user-defined targets defined in the .msp library
- Aligned annotations are consistent with the QC metrics Rapid QC-MS reports (instrument sensitivity, retention time shifts, mass accuracy)
- ES and RS library alignments completed as separate modules, each with its matrix-matched .msp library

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may have processing bugs
- MS-DIAL ver.4.9.22121823 and MSConvert must be downloaded and installed manually — they are not installed by Rapid QC-MS setup
- ES and RS are often not from the same biological matrix, so a single shared library cannot serve both workflows; using the wrong matrix-matched library will misalign targets

## Evidence

- [methods] MS-DIAL then processes the mzML files, carrying out quantification and annotation procedures for all user-defined targets.: [evidence span withheld — anchor status: not_found]
- [other] the key distinction is the molecular targets library, since ES and RS are often not from the same biological matrix, allowing searches for different molecular targets corresponding to matrix type.: [evidence span withheld — anchor status: not_found]
- [other] Match each ES feature's m/z and MS/MS or EI spectrum against the .msp library entries using spectral similarity scoring in MS-DIAL: [evidence span withheld — anchor status: not_found]
- [other] combine analyte name, m/z, RT, and MS2 information in a .msp-format file which they upload into Rapid QC-MS (Settings > Biological Standards).: [evidence span withheld — anchor status: not_found]
