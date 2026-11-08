---
name: lc-im-msms-feature-extraction
description: Use when processing per-sample LC-IM-MS/MS ion-mobility raw files of
  N-Me derivatized unsaturated sterols and you need 4D feature extraction plus per-sample
  isomer-level sterol identifications.
when_to_use_negative:
- Raw files are not N-Me derivatized unsaturated sterol lipids acquired by LC-IM-MS/MS
  — the matching dimensions (CCS, N-Me fragment patterns) are specific to this derivatization
  and ion-mobility acquisition
- You only need MS/MS fragment generation or CCS model training (parts 1 and 2 of
  QCCAssisted4DSterol) rather than per-sample data processing and identification
- No 4D sterol lipid database is available yet — build the database (m/z, RT, CCS,
  fragment prediction) first
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Python
  role: Implementation language of the part 3 data-processing and sterol-identification
    scripts that extract 4D features and match samples against the 4D database
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
- name: Jupyter Notebook
  role: All functions of the data processing and sterol identification workflow are
    implemented in Jupyter notebooks
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/anie.202507483
    title: Quantum Chemistry Calculation‐Assisted Large‐Scale Collision Cross Section
      Prediction Empowers Derivatization‐Enhanced Multidimensional Metabolomics
  source_content_hash: sha256:09c5e2c0ad3382184d6e0a7fded260ad06969c2dc6b43c1d13efa45c07d428f8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Chen-micslab/QCCAssisted4DSterol
  tools:
  - Jupyter Notebook
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lc-im-msms-feature-extraction

## Summary

Extracts 4D features (m/z, retention time, collision cross section, MS/MS fragments) from per-sample LC-IM-MS/MS sterolomics raw files using the Python/Jupyter data-processing notebooks of QCCAssisted4DSterol, then matches them against the 4D derivatized sterol lipid database (n = 4891) for isomer-level sterol identification.

## When to use

Use when processing per-sample LC-IM-MS/MS ion-mobility raw files of N-Me derivatized unsaturated sterols and you need 4D feature extraction plus per-sample isomer-level sterol identifications.

## When NOT to use

- Raw files are not N-Me derivatized unsaturated sterol lipids acquired by LC-IM-MS/MS — the matching dimensions (CCS, N-Me fragment patterns) are specific to this derivatization and ion-mobility acquisition
- You only need MS/MS fragment generation or CCS model training (parts 1 and 2 of QCCAssisted4DSterol) rather than per-sample data processing and identification
- No 4D sterol lipid database is available yet — build the database (m/z, RT, CCS, fragment prediction) first

## Inputs

- Per-sample LC-IM-MS/MS raw files of derivatized (N-Me) sterol lipids, organized one folder per sample under ./Search/tissue
- 4D sterol lipid database of derivatized sterols (n = 4891) with m/z, RT, CCS, and fragment ion entries

## Outputs

- One identification result table per sample folder under ./Search/tissue, listing matched sterol identifications
- Per-match 4D evidence fields: m/z, retention time, collision cross section, and MS/MS fragment-ion evidence

## How to apply

Organize per-sample LC-IM-MS/MS raw files under the repository's `./Search/tissue` folder structure, where each sample has its own folder. Run the part 3 data-processing Jupyter notebooks (Python) to extract 4D features — precursor m/z, retention time, collision cross section, and MS/MS fragment ions — from each sample. Match each sample's 4D features against the 4D sterol lipid database of derivatized sterols (n = 4891) built in the preceding step, using all four dimensions (m/z, RT, CCS, N-Me fragment ion patterns) as matching evidence rather than m/z alone. Assign isomer-level sterol lipid identifications per sample based on the 4D matches, and write one identification result table per sample recording matched sterol entries with their 4D evidence. This is the third part of the QCCAssisted4DSterol workflow, after MS/MS calculation for N-Me derived unsaturated sterol lipids and QCC-assisted CCS prediction.

## Related tools

- **Python** (Implementation language of the part 3 data-processing and sterol-identification scripts that extract 4D features and match samples against the 4D database) — https://github.com/Chen-micslab/QCCAssisted4DSterol
- **Jupyter Notebook** (All functions of the data processing and sterol identification workflow are implemented in Jupyter notebooks) — https://github.com/Chen-micslab/QCCAssisted4DSterol

## Examples

```
cd QCCAssisted4DSterol && jupyter notebook  # open the part 3 data-processing notebooks; sample files live in ./Search/tissue
```

## Evaluation signals

- An identification result table is produced for every sample folder under ./Search/tissue — no sample folder is left without output
- Each output table contains matched sterol identifications with all four evidence fields: m/z, RT, CCS, and fragment-ion evidence
- Identified sterols are isomer-level assignments of N-Me derivatized unsaturated sterols, consistent with the 4D database (n = 4891) entries
- Identifications support tissue-specific distribution patterns of over 100 sterol lipids as reported in the paper's high-coverage unsaturated sterolomics

## Limitations

- MS/MS fragment generation code was only tested on sterol lipids, though theoretically applicable to all molecules containing C=C bonds — treat identifications of other lipid classes as unvalidated
- Sample file organization must follow the per-sample folder structure under ./Search/tissue; deviations may break the processing notebooks
- Matching depends on the preceding QCC-assisted CCS prediction and RT/fragment prediction steps; errors in the 4D database propagate into identifications

## Evidence
