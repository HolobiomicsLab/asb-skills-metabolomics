---
name: four-dimensional-library-matching
description: Use when per-sample LC-IM-MS/MS raw files with ion-mobility data must
  be matched against a pre-built 4D database to produce isomer-level sterol lipid
  identifications with m/z, RT, CCS, and fragment-ion evidence.
when_to_use_negative:
- The target molecules are not sterol lipids containing C=C bonds — the MS/MS generation
  is only tested on sterol lipids even though it is theoretically applicable to all
  molecules with a C=C bond
- No ion-mobility (CCS) dimension is available in the input data, so only m/z- and
  RT-based library search is possible
- No pre-built 4D derivatized sterol database exists for the compounds of interest;
  build the database first via MS/MS calculation and CCS/RT prediction workflows
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Python
  role: Language for the part 3 data-processing scripts that extract 4D features and
    match samples against the 4D sterol database
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
- name: Jupyter Notebook
  role: All functions of the data processing and sterol identification workflow are
    implemented in Jupyter notebooks
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
- name: RDkit
  role: Underlies the MS/MS fragment-pattern generation of the N-Me derived sterol
    database entries used as a matching dimension
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/anie.202507483
    title: Quantum Chemistry Calculation‐Assisted Large‐Scale Collision Cross Section
      Prediction Empowers Derivatization‐Enhanced Multidimensional Metabolomics
  source_content_hash: sha256:0a85b813bfb3e794f0b8a7194cfdefdb655e6012bf5ca79167f11a7f37310ed8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Chen-micslab/QCCAssisted4DSterol
  tools:
  - Jupyter Notebook
  - Python
  - RDkit
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# four-dimensional-library-matching

## Summary

Matches per-sample LC-IM-MS/MS sterolomics features against a 4D sterol lipid database (n = 4891 derivatized sterols) using m/z, retention time, collision cross section, and N-Me fragment ion patterns as matching dimensions to assign isomer-level sterol lipid identifications per tissue sample.

## When to use

Use when per-sample LC-IM-MS/MS raw files with ion-mobility data must be matched against a pre-built 4D database to produce isomer-level sterol lipid identifications with m/z, RT, CCS, and fragment-ion evidence.

## When NOT to use

- The target molecules are not sterol lipids containing C=C bonds — the MS/MS generation is only tested on sterol lipids even though it is theoretically applicable to all molecules with a C=C bond
- No ion-mobility (CCS) dimension is available in the input data, so only m/z- and RT-based library search is possible
- No pre-built 4D derivatized sterol database exists for the compounds of interest; build the database first via MS/MS calculation and CCS/RT prediction workflows

## Inputs

- Per-sample LC-IM-MS/MS raw files organized in the per-sample folder structure under ./Search/tissue
- 4D sterol lipid database of derivatized sterols (n = 4891) combining m/z, RT, CCS, and fragment ion predictions

## Outputs

- One identification result table per sample, recording matched sterol entries with m/z, RT, CCS, and fragment-ion evidence fields
- Isomer-level sterol lipid identifications per tissue sample

## How to apply

Load per-sample raw LC-IM-MS/MS files from the repository's per-sample folder structure under ./Search/tissue, then execute the part 3 data-processing Jupyter notebooks in Python to extract 4D features (m/z, retention time, collision cross section, MS/MS fragments) from each sample. Match each sample's 4D features against the task_004 4D sterol lipid database of derivatized sterols (n = 4891), which combines retention time and fragment ion prediction with predicted CCS, using m/z, RT, CCS, and N-Me fragment ion patterns as matching dimensions. Assign isomer-level sterol lipid identifications for each tissue sample based on the 4D matches, and write one identification result table per sample recording the matched sterol entries and their 4D evidence. The rationale for four dimensions is that the C=C bond-targeted N-Me derivatization plus QCC-assisted CCS prediction resolves unsaturated sterol isomers that single-dimension matching cannot separate.

## Related tools

- **Python** (Language for the part 3 data-processing scripts that extract 4D features and match samples against the 4D sterol database) — https://github.com/Chen-micslab/QCCAssisted4DSterol
- **Jupyter Notebook** (All functions of the data processing and sterol identification workflow are implemented in Jupyter notebooks) — https://github.com/Chen-micslab/QCCAssisted4DSterol
- **RDkit** (Underlies the MS/MS fragment-pattern generation of the N-Me derived sterol database entries used as a matching dimension) — https://github.com/Chen-micslab/QCCAssisted4DSterol

## Examples

```
Run the part 3 data-processing Jupyter notebooks on the per-sample files in ./Search/tissue: `jupyter notebook ./Search/tissue`
```

## Evaluation signals

- An identification result table is produced for every sample folder under ./Search/tissue
- Each identification result table contains the matched sterol identifications with their m/z, RT, CCS, and fragment-ion evidence fields
- Matches are supported on all four dimensions (m/z, retention time, collision cross section, N-Me fragment ion patterns), not just precursor m/z
- Identification reaches isomer level — tissue-specific distribution patterns of over 100 sterol lipids are quantitatively resolved, consistent with the reported high-coverage unsaturated sterolomics

## Limitations

- MS/MS fragment-pattern generation is theoretically applicable to all molecules containing a C=C bond but was only tested on sterol lipids
- Matching depends on the pre-built derivatized sterol database (n = 4891); compounds absent from the database cannot be identified
- Requires ion-mobility LC-IM-MS/MS acquisition so that CCS is available as a matching dimension

## Evidence
