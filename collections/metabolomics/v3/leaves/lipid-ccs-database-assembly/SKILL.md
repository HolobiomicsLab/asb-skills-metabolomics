---
name: lipid-ccs-database-assembly
description: Reference for building or auditing an experimental lipid CCS/m/z reference
  database from cleaned per-instrument CCS collections (DTIM, TWIM, TIMS) when per-field
  totals must be verified against reported row counts.
when_to_use_negative:
- The goal is CCS prediction for lipids absent from experimental collections — use
  the trained machine-learning prediction models and the predicted database (145,388
  entries) instead of re-assembling the experimental database.
- The analysis target is lipidomics data analysis (statistics, plotting, identification)
  on an IM-MS dataset — use the lipydomics stats/plotting/identification modules on
  a Dataset instance rather than rebuilding the reference database.
- Input collections are not cleaned or lack per-entry ionization-mode/drift-tube-type
  annotations, so per-field totals cannot be validated.
tools:
- name: LiPydomics
  role: Package whose repository contains the cleaned per-dataset JSON CCS collections
    and database build scripts used to assemble and store the combined experimental
    lipid database
  repo: https://github.com/dylanhross/lipydomics
- name: Python
  role: Language in which the build scripts and the lipydomics package are implemented;
    used to run aggregation and SQLite queries
  repo: https://github.com/dylanhross/lipydomics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:76a53f8bf573534fb4e26a01ad71bb062f54c1d38faa1631724a4d1e17a62080
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# lipid-ccs-database-assembly

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assemble a combined experimental lipid reference database by aggregating cleaned per-dataset JSON CCS collections (DTIM, TWIM, TIMS) into a single SQLite file with m/z, CCS, lipid class, ionization mode, and drift-tube-type fields, then verify per-field totals. This provides the verifiable experimental backbone that LiPydomics uses for lipid identification and for training CCS/retention-time prediction models.

## When to use

Reference for building or auditing an experimental lipid CCS/m/z reference database from cleaned per-instrument CCS collections (DTIM, TWIM, TIMS) when per-field totals must be verified against reported row counts.

## When NOT to use

- The goal is CCS prediction for lipids absent from experimental collections — use the trained machine-learning prediction models and the predicted database (145,388 entries) instead of re-assembling the experimental database.
- The analysis target is lipidomics data analysis (statistics, plotting, identification) on an IM-MS dataset — use the lipydomics stats/plotting/identification modules on a Dataset instance rather than rebuilding the reference database.
- Input collections are not cleaned or lack per-entry ionization-mode/drift-tube-type annotations, so per-field totals cannot be validated.

## Inputs

- cleaned per-dataset JSON CCS collections (DTIM, TWIM, and TIMS experimental measurements)
- database build scripts from the lipydomics package repository (github.com/dylanhross/lipydomics)

## Outputs

- assembled SQLite experimental lipid database with m/z, CCS, lipid class, ionization mode, and drift-tube-type fields per entry
- verified summary counts: total entries (7907), distinct lipid classes (45), positive/negative mode totals (5110/2797), DTIM/TWIM/TIMS totals (1285/596/6026)

## How to apply

Locate the cleaned per-dataset JSON CCS collections and the database build scripts included in the lipydomics package repository. Run the build scripts to load each cleaned JSON collection and aggregate its entries into a single combined experimental lipid database, storing the result as a SQLite file with fields for m/z, CCS, lipid class, ionization mode, and drift-tube type for each entry. Then query the assembled SQLite database to count total entries, per-mode totals (positive/negative), per-drift-tube-type totals (DTIM/TWIM/TIMS), and distinct lipid classes. Validate against the reported reference totals: 7907 experimental CCS entries across 45 lipid classes, 5110 positive-mode and 2797 negative-mode measurements, and 1285 DTIM, 596 TWIM, and 6026 TIMS entries. The rationale is that experimental collections do not yet comprehensively cover the lipid chemical space, so a verified combined database is the prerequisite for both Level 3 lipid identification and downstream machine-learning CCS prediction.

## Related tools

- **LiPydomics** (Package whose repository contains the cleaned per-dataset JSON CCS collections and database build scripts used to assemble and store the combined experimental lipid database) — https://github.com/dylanhross/lipydomics
- **Python** (Language in which the build scripts and the lipydomics package are implemented; used to run aggregation and SQLite queries) — https://github.com/dylanhross/lipydomics

## Evaluation signals

- Total row count of the assembled SQLite database equals 7907 experimental CCS entries.
- Distinct lipid-class count equals 45, matching the reported class coverage (with 23 classes containing HILIC retention times).
- Ionization-mode split matches 5110 positive-mode and 2797 negative-mode measurements.
- Drift-tube-type split matches 1285 DTIM, 596 TWIM, and 6026 TIMS entries.
- Every row has non-null m/z, CCS, lipid class, ionization mode, and drift-tube-type fields.

## Limitations

- The assembled experimental database covers only lipids present in published experimental CCS collections; coverage of bacterial lipids is limited and the broader chemical space is addressed by machine-learning prediction instead.
- The database is built to be highly extensible and customizable so it can be expanded as more data becomes available — re-running assembly after adding collections will change the totals from the reference values.
- A missing repository link in the discussion section suggests the database and prediction models ship with the package, so verification depends on the versions of the cleaned JSON collections in the repository.

## Evidence
