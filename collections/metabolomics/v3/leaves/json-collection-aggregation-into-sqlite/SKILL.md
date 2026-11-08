---
name: json-collection-aggregation-into-sqlite
description: Use when cleaned per-dataset JSON collections of experimental CCS values
  must be merged into one queryable reference database with per-field totals that
  can be validated against reported row counts.
when_to_use_negative:
- The goal is CCS/retention-time prediction from trained machine-learning models rather
  than assembling the experimental reference database.
- Input is raw IM-MS lipidomics data requiring statistical or multivariate analysis
  (use the lipydomics stats/plotting/identification modules instead).
- The source collections are not cleaned JSON CCS collections with the required fields
  (m/z, CCS, lipid class, ionization mode, drift-tube type).
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LiPydomics
  role: Provides the cleaned JSON CCS collections and the database build scripts that
    aggregate entries into the combined experimental lipid database
  repo: https://github.com/dylanhross/lipydomics
- name: Python
  role: Language used to run the build scripts and to load/aggregate JSON entries
    into the SQLite database file
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:2984c63d1e054a209407cc94dd248d9e6c42c2e0a3c49a3aacda6a58bb8cfb0c
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# json-collection-aggregation-into-sqlite

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles cleaned per-dataset JSON CCS collections into a single combined experimental lipid reference database stored as a SQLite file, with fields for m/z, CCS, lipid class, ionization mode, and drift-tube type. This produces a verifiable, queryable experimental CCS reference (7907 entries, 45 lipid classes) underlying LiPydomics identification.

## When to use

Use when cleaned per-dataset JSON collections of experimental CCS values must be merged into one queryable reference database with per-field totals that can be validated against reported row counts.

## When NOT to use

- The goal is CCS/retention-time prediction from trained machine-learning models rather than assembling the experimental reference database.
- Input is raw IM-MS lipidomics data requiring statistical or multivariate analysis (use the lipydomics stats/plotting/identification modules instead).
- The source collections are not cleaned JSON CCS collections with the required fields (m/z, CCS, lipid class, ionization mode, drift-tube type).

## Inputs

- cleaned per-dataset JSON collections of experimental lipid CCS values
- database build scripts from the lipydomics package repository

## Outputs

- combined experimental lipid database stored as a SQLite file with fields for m/z, CCS, lipid class, ionization mode, and drift-tube type
- row-count and per-field summary statistics (total entries, per-mode totals, per-drift-tube-type totals, distinct lipid class count)

## How to apply

Locate the cleaned per-dataset JSON CCS collections and the database build scripts included in the lipydomics package repository. Run the build scripts to load each cleaned JSON collection and aggregate its entries into a single combined experimental lipid database, stored as a SQLite file with fields for m/z, CCS, lipid class, ionization mode, and drift-tube type for each entry. Query the assembled SQLite database to count total entries, per-mode totals (positive/negative), per-drift-tube-type totals (DTIM/TWIM/TIMS), and distinct lipid classes. Validate that counts match the reported totals: 7907 experimental CCS entries across 45 lipid classes, 5110 positive and 2797 negative mode, and 1285 DTIM, 596 TWIM, 6026 TIMS entries. The rationale is that experimental CCS collections alone do not comprehensively cover the lipid chemical space, so a single consolidated database is the substrate for both identification and downstream machine-learning CCS prediction.

## Related tools

- **LiPydomics** (Provides the cleaned JSON CCS collections and the database build scripts that aggregate entries into the combined experimental lipid database) — https://github.com/dylanhross/lipydomics
- **Python** (Language used to run the build scripts and to load/aggregate JSON entries into the SQLite database file)

## Evaluation signals

- Total SQLite row count equals 7907 experimental CCS entries
- Number of distinct lipid classes in the database equals 45
- Ionization-mode split is 5110 positive-mode and 2797 negative-mode entries
- Drift-tube-type split is 1285 DTIM, 596 TWIM, and 6026 TIMS entries
- Every SQLite row carries populated fields for m/z, CCS, lipid class, ionization mode, and drift-tube type

## Limitations

- The assembled database reflects only the experimental CCS collections available at build time; the package notes the database is extensible and can be expanded as more data becomes available.
- Coverage is limited to the 45 lipid classes in the assembled experimental collections; comprehensive class-level coverage requires supplementing with the predicted lipid database (145,388 entries).

## Evidence
