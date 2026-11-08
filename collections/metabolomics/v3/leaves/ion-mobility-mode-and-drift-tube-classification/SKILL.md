---
name: ion-mobility-mode-and-drift-tube-classification
description: Use when assembling or auditing a lipid CCS reference database from multiple
  experimental ion-mobility collections and you must classify entries by ionization
  mode and drift-tube type to report coverage totals.
when_to_use_negative:
- Input entries are predicted (ML-generated) CCS values rather than experimental measurements
  — the per-mode/per-platform totals here refer to the 7907-value experimental database,
  not the 145,388-entry predicted database.
- The CCS collection lacks ionization-mode or drift-tube metadata, so entries cannot
  be classified as positive/negative or DTIM/TWIM/TIMS without additional provenance
  lookup.
- The target lipids fall outside the 45 lipid classes covered by the assembled experimental
  reference database.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LiPydomics
  role: Provides the cleaned per-dataset JSON CCS collections and the database build
    scripts used to aggregate entries into the combined SQLite experimental lipid
    database.
  repo: https://github.com/dylanhross/lipydomics
- name: Python
  role: Execution environment for running the build scripts, loading the JSON collections,
    and querying the assembled SQLite database for per-field counts.
  repo: https://github.com/dylanhross/lipydomics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:aa177aae3fc040ea4226980a4875f3b11a6fb953a3e3503077e4af61f8543239
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dylanhross/lipydomics
  repo_url: https://github.com/dylanhross/lipydomics
  tools:
  - LiPydomics
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ion-mobility-mode-and-drift-tube-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tag each experimental lipid CCS entry with its ionization mode (positive/negative) and drift-tube type (DTIM, TWIM, or TIMS) while aggregating cleaned per-dataset CCS collections into the LiPydomics combined reference lipid database, enabling per-mode and per-platform coverage audits.

## When to use

Use when assembling or auditing a lipid CCS reference database from multiple experimental ion-mobility collections and you must classify entries by ionization mode and drift-tube type to report coverage totals.

## When NOT to use

- Input entries are predicted (ML-generated) CCS values rather than experimental measurements — the per-mode/per-platform totals here refer to the 7907-value experimental database, not the 145,388-entry predicted database.
- The CCS collection lacks ionization-mode or drift-tube metadata, so entries cannot be classified as positive/negative or DTIM/TWIM/TIMS without additional provenance lookup.
- The target lipids fall outside the 45 lipid classes covered by the assembled experimental reference database.

## Inputs

- cleaned per-dataset JSON CCS collections (experimental CCS measurements from DTIM, TWIM, and TIMS platforms)
- lipydomics database build scripts from the GitHub repository
- existing partial SQLite lipid database (for incremental extension)

## Outputs

- combined experimental reference lipid database as a SQLite file with per-entry fields for m/z, CCS, lipid class, ionization mode, and drift-tube type
- coverage summary counts: total entries, positive/negative mode totals, DTIM/TWIM/TIMS totals, and distinct lipid-class count

## How to apply

Locate the cleaned per-dataset JSON CCS collections shipped in the lipydomics repository and run the database build scripts to aggregate their entries into a single experimental lipid database stored as SQLite. For each entry, carry forward and normalize the fields for m/z, CCS, lipid class, ionization mode, and drift-tube type, so every row is explicitly tagged as positive- or negative-mode and as DTIM, TWIM, or TIMS. Rationale: the source collections come from different IM-MS platforms (drift-tube, traveling-wave, and trapped ion mobility spectrometry) and both ion polarities, and the assembled database's coverage is only verifiable if these per-field classifications are preserved. After the build, query the SQLite file to count total entries, distinct lipid classes, per-mode totals, and per-drift-tube-type totals. Validate against the reported reference totals: 7907 CCS values across 45 lipid classes, 5110 positive-mode and 2797 negative-mode measurements, and 1285 DTIM, 596 TWIM, and 6026 TIMS entries.

## Related tools

- **LiPydomics** (Provides the cleaned per-dataset JSON CCS collections and the database build scripts used to aggregate entries into the combined SQLite experimental lipid database.) — https://github.com/dylanhross/lipydomics
- **Python** (Execution environment for running the build scripts, loading the JSON collections, and querying the assembled SQLite database for per-field counts.) — https://github.com/dylanhross/lipydomics

## Examples

```
SELECT ionization_mode, COUNT(*) FROM lipid_ccs GROUP BY ionization_mode; SELECT drift_tube_type, COUNT(*) FROM lipid_ccs GROUP BY drift_tube_type;
```

## Evaluation signals

- Total row count of the assembled SQLite database equals 7907 experimental CCS entries.
- Per-mode counts are 5110 positive and 2797 negative, and 5110 + 2797 = 7907.
- Per-drift-tube-type counts are 1285 DTIM, 596 TWIM, and 6026 TIMS, summing to 7907.
- COUNT(DISTINCT lipid_class) over the database equals 45.
- Schema check: every populated row carries non-null m/z, CCS, lipid class, ionization mode, and drift-tube type values.

## Limitations

- The experimental reference database is limited to 45 lipid classes; collections do not yet comprehensively cover the vast lipid chemical space, which is why prediction models and the 145,388-entry predicted database were built.
- Coverage is heterogeneous across platforms (TIMS contributes 6026 of 7907 values), so per-platform sub-databases may be sparse for some classes and modes.
- The package and database are extensible, so re-running builds with newly added collections will change the reference totals away from the reported 7907/45 values.

## Evidence
