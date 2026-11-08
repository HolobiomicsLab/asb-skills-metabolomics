---
name: metabolite-mz-matching
description: Use when annotating peaks from an untargeted LC-MS metabolomics feature
  table (m/z per sample) with optional MS2 spectra in .mgf/.mzML format, and you need
  metabolite candidates from CEU Mass Mediator plus spectral-library confirmation.
when_to_use_negative:
- Input is raw vendor LC-MS data rather than a preprocessed feature table — BiomiX
  expects m/z-per-sample feature tables, not raw spectra conversion
- The omics type is not metabolomics (e.g. transcriptomics, methylomics) — other BiomiX
  modules (DESeq2/Limma, ChAMP) handle those, not the CEU Mass Mediator matching
- Instrument-level QC normalization is required — BiomiX does not support metabolomics
  instrument normalization; QC samples can only be previewed and removed
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: BiomiX
  role: GUI pipeline orchestrating the metabolomics annotation module (m/z matching
    and MS2 spectral annotation)
  repo: https://github.com/IxI-97/BiomiX
- name: CEU Mass Mediator
  role: Database queried via the CMMR R package for MS1-level m/z matching of features
    (default 15-ppm error)
- name: TidyMass
  role: R package used to match MS2 fragmentation spectra against HMDB, MoNA, and
    MassBank libraries
- name: R
  role: Underlying runtime environment for BiomiX and its annotation packages (CMMR,
    TidyMass)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:5bfd9883bc086f5908f6c9bd6db3814dd70c0ca6ed58eab8c895f8dc30c3e633
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/IxI-97/BiomiX
  repo_url: https://github.com/IxI-97/BiomiX
  tools:
  - BiomiX
  - CEU Mass Mediator
  - R
  - TidyMass
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# metabolite-mz-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates untargeted LC-MS metabolomics peaks by matching feature m/z values against the CEU Mass Mediator database via the CMMR R package (default 15-ppm tolerance), then refining annotations with MS2 fragmentation spectra through TidyMass, where MS2-confirmed hits override MS1-only annotations.

## When to use

Use when annotating peaks from an untargeted LC-MS metabolomics feature table (m/z per sample) with optional MS2 spectra in .mgf/.mzML format, and you need metabolite candidates from CEU Mass Mediator plus spectral-library confirmation.

## When NOT to use

- Input is raw vendor LC-MS data rather than a preprocessed feature table — BiomiX expects m/z-per-sample feature tables, not raw spectra conversion
- The omics type is not metabolomics (e.g. transcriptomics, methylomics) — other BiomiX modules (DESeq2/Limma, ChAMP) handle those, not the CEU Mass Mediator matching
- Instrument-level QC normalization is required — BiomiX does not support metabolomics instrument normalization; QC samples can only be previewed and removed

## Inputs

- LC-MS untargeted metabolomics feature table (m/z values per sample)
- MS2 fragmentation spectra in .mgf format (preferred) or .mzML
- User-selected adduct set and ionization mode (positive/neutral/negative)

## Outputs

- metabolomics_annotations.csv with fields for feature m/z, assigned metabolite, database source, and annotation level (MS1 vs MS2)
- MS1-level candidate metabolite lists from CEU Mass Mediator
- MS2-confirmed annotations from HMDB/MoNA/MassBank spectral matches

## How to apply

Load the LC-MS feature table (m/z values per sample) and the MS2 spectra into BiomiX's metabolomics module, preferring .mgf over .mzML because .mgf files are lighter and optimize RAM usage with many files. Match each feature's m/z against the CEU Mass Mediator database through the CMMR R package using the default 15-ppm mass tolerance and the user-selected adduct set (positive mode default; neutral and negative modes also available) to generate MS1-level candidate metabolite lists. Then match MS2 fragmentation spectra against the HMDB, MoNA, and MassBank spectral libraries using TidyMass, resolving conflicts with the priority order HMDB > MoNA > MassBank. Wherever an MS2 spectral match exists for a precursor, automatically replace the MS1-only annotation because MS2 is more reliable. Finally, filter remaining candidates using HMDB specimen-occurrence information to retain only biologically plausible metabolites.

## Related tools

- **BiomiX** (GUI pipeline orchestrating the metabolomics annotation module (m/z matching and MS2 spectral annotation)) — https://github.com/IxI-97/BiomiX
- **CEU Mass Mediator** (Database queried via the CMMR R package for MS1-level m/z matching of features (default 15-ppm error))
- **TidyMass** (R package used to match MS2 fragmentation spectra against HMDB, MoNA, and MassBank libraries)
- **R** (Underlying runtime environment for BiomiX and its annotation packages (CMMR, TidyMass))

## Evaluation signals

- metabolomics_annotations.csv exists and contains fields for feature m/z, assigned metabolite, database source, and annotation level (MS1 vs MS2)
- All reported m/z matches fall within the 15-ppm mass tolerance
- Every feature with an available MS2 spectral match carries an MS2-level annotation that replaced the MS1-only annotation
- Conflicting MS2 library hits are resolved with the documented priority HMDB > MoNA > MassBank
- Remaining candidates are filtered against HMDB specimen occurrence so only biologically plausible metabolites are retained

## Limitations

- BiomiX does not support instrument normalization for metabolomics; QC samples can only be previewed (PCA/UMAP via 'QC' labels in the CONDITION column) and removed automatically afterwards
- The Mac version has installation issues; Linux or Windows are recommended for running the metabolomics module
- Users are strongly recommended to carefully review the dataset before using BiomiX; no automated validation of inputs is performed

## Evidence
