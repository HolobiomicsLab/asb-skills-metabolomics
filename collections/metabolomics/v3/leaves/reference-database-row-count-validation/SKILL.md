---
name: reference-database-row-count-validation
description: Use when reproducing or auditing the assembled experimental lipid CCS
  reference database (SQLite) and its row counts — per-mode (positive/negative), per-drift-tube-type
  (DTIM/TWIM/TIMS), and per-lipid-class coverage.
when_to_use_negative:
- When the goal is CCS prediction with the trained ML models (e.g. the 145,388-entry
  predicted lipid database) rather than validating the experimental reference database
  totals.
- When the input is an unnormalized or raw IM-MS feature list rather than the cleaned
  per-dataset JSON CCS collections the build scripts expect.
- When only approximate database coverage is needed for reporting — this skill is
  a strict reproduction/audit procedure with exact acceptance counts.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LiPydomics
  role: Provides the cleaned per-dataset JSON CCS collections, the database build
    scripts, and the SQLite experimental lipid database being assembled and count-validated
  repo: https://github.com/dylanhross/lipydomics
- name: Python
  role: Execution environment for running the database build scripts and issuing SQLite
    count queries (e.g. via the sqlite3 module)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:7b90df5d6caa20119c16c5aaee805fc5c70d74a5e174245b671a7da1326a5dc8
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# reference-database-row-count-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces and validates the LiPydomics experimental reference lipid database by building a SQLite store from the cleaned per-dataset JSON CCS collections and verifying row and per-field totals (7907 CCS values across 45 lipid classes) against published figures.

## When to use

Use when reproducing or auditing the assembled experimental lipid CCS reference database (SQLite) and its row counts — per-mode (positive/negative), per-drift-tube-type (DTIM/TWIM/TIMS), and per-lipid-class coverage.

## When NOT to use

- When the goal is CCS prediction with the trained ML models (e.g. the 145,388-entry predicted lipid database) rather than validating the experimental reference database totals.
- When the input is an unnormalized or raw IM-MS feature list rather than the cleaned per-dataset JSON CCS collections the build scripts expect.
- When only approximate database coverage is needed for reporting — this skill is a strict reproduction/audit procedure with exact acceptance counts.

## Inputs

- cleaned per-dataset JSON CCS collections from the lipydomics repository
- database build scripts included in the lipydomics package
- expected totals from the published reference: 7907 CCS values, 45 lipid classes, 5110/2797 positive/negative, 1285 DTIM / 596 TWIM / 6026 TIMS

## Outputs

- assembled SQLite experimental lipid database with per-entry m/z, CCS, lipid class, ionization mode, and drift-tube type fields
- row-count audit report: total entries, per-mode totals, per-drift-tube-type totals, and distinct lipid-class count

## How to apply

Locate the cleaned per-dataset JSON CCS collections and the database build scripts in the lipydomics package repository (github.com/dylanhross/lipydomics), then run the build scripts to load each JSON collection and aggregate all entries into a single combined experimental lipid database. Store the result as a SQLite file whose schema carries m/z, CCS, lipid class, ionization mode, and drift-tube type fields for every entry. Query the assembled SQLite database with COUNT(*) and COUNT(DISTINCT lipid_class) queries to compute total entries, positive/negative-mode totals, DTIM/TWIM/TIMS totals, and the number of distinct lipid classes. Treat the published totals as acceptance criteria: 7907 experimental CCS entries across 45 lipid classes, 5110 positive and 2797 negative mode, and 1285 DTIM, 596 TWIM, 6026 TIMS entries — any mismatch indicates a build, load, or deduplication error.

## Related tools

- **LiPydomics** (Provides the cleaned per-dataset JSON CCS collections, the database build scripts, and the SQLite experimental lipid database being assembled and count-validated) — https://github.com/dylanhross/lipydomics
- **Python** (Execution environment for running the database build scripts and issuing SQLite count queries (e.g. via the sqlite3 module))

## Evaluation signals

- Total row count in the assembled SQLite database equals exactly 7907 experimental CCS entries.
- COUNT(DISTINCT lipid_class) equals exactly 45.
- Positive-mode total (5110) plus negative-mode total (2797) equals the overall total of 7907 — a mode-partition invariant.
- DTIM (1285) + TWIM (596) + TIMS (6026) sums to 7907 — a drift-tube-type partition invariant.
- Per-field totals can be reproduced by rerunning the build scripts from the cleaned JSON collections, confirming the database is verifiable from its sources.

## Limitations

- Coverage mirrors the experimental CCS collections the package was built from, which 'do not yet comprehensively cover the vast lipid chemical space'; later data updates would shift the acceptance counts away from the published totals.
- The database carries lipid class/subclass and fatty-acyl sum composition level information (HILIC-IM-MS Level 3 identifications), so field-level validation cannot confirm species-level structural annotations.
- Only 23 of the 45 lipid classes contain HILIC retention-time information, so retention-time field coverage cannot be validated for all classes.

## Evidence
