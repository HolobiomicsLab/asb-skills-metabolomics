---
name: lipidomics-database-construction-sqlite
description: Use when building a broad-coverage lipid identification database from
  scratch — enumerating lipid species by class, chain length (10–30 carbons, even/odd)
  and unsaturation (0–6) — to back IM-MS/HILIC lipid identification with predicted
  CCS and RT values.
when_to_use_negative:
- When only experimental CCS/RT measurements are needed for a small set of known lipids
  — the experimental database of m/z and CCS of 45 lipid classes (23 with HILIC RTs)
  already serves that purpose
- When the target analysis is statistical/multivariate analysis or plotting of an
  existing lipidomics dataset (the 'stats', 'plotting', or 'interactive' modules are
  the right entry points, not database construction)
- When species-level (not sum-composition) lipid structures are required — the enumeration
  covers fatty acyl sum compositions at Level 3 identification depth
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0218
tools:
- name: LiPydomics
  role: Source of lipid class definitions, identification-module adduct lists, and
    the target SQLite/JSON database schema
  repo: https://github.com/dylanhross/lipydomics
- name: Python
  role: Implementation language for the enumeration and database-writing script
- name: scikit-learn
  role: Provides the trained CCS and HILIC retention time regression models used to
    populate class-specific predictions
- name: SciPy
  role: Scientific computing support for the prediction/database workflow
- name: LipidCCS
  role: Feature scheme reference for the chain-length/unsaturation CCS prediction
    models
- name: LipidMass
  role: Source of lipid class building-block metadata used to define the enumeration
    space
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:3fda9c3774dee8236bf3f85e84876ed7ae79b3671ec746c2318effe7a3c981e5
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
  - LipidCCS
  - LipidMass
  - Python
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lipidomics-database-construction-sqlite

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematically enumerate a comprehensive predicted lipid database by combining fatty acyl chain compositions (10–30 carbons, even and odd; 0–6 degrees of unsaturation) across LipidMass-defined lipid classes, then populate theoretical m/z/adduct, predicted CCS, and predicted HILIC retention time entries into SQLite/JSON tables, as done in LiPydomics (145,388 entries).

## When to use

Use when building a broad-coverage lipid identification database from scratch — enumerating lipid species by class, chain length (10–30 carbons, even/odd) and unsaturation (0–6) — to back IM-MS/HILIC lipid identification with predicted CCS and RT values.

## When NOT to use

- When only experimental CCS/RT measurements are needed for a small set of known lipids — the experimental database of m/z and CCS of 45 lipid classes (23 with HILIC RTs) already serves that purpose
- When the target analysis is statistical/multivariate analysis or plotting of an existing lipidomics dataset (the 'stats', 'plotting', or 'interactive' modules are the right entry points, not database construction)
- When species-level (not sum-composition) lipid structures are required — the enumeration covers fatty acyl sum compositions at Level 3 identification depth

## Inputs

- LiPydomics lipid class definitions and LipidMass-based building-block metadata (31 lipid classes) from the package's identification module
- Trained scikit-learn CCS prediction models for 22 lipid classes (LipidCCS-style features: fatty acyl chain length, unsaturation)
- Trained scikit-learn HILIC retention time regression models for 23 lipid classes
- LiPydomics documented ESI adduct lists (e.g., [M+H]+, [M+Na]+, [M-H]-)

## Outputs

- Predicted lipid database with 145,388 lipid species entries (SQLite/JSON tables per the package's database schema)
- 94,451 predicted CCS values covering 22 lipid classes
- 106,020 predicted HILIC retention time values covering 23 lipid classes
- Per-species theoretical m/z and adduct annotations

## How to apply

Load the LiPydomics lipid class definitions and LipidMass-based building-block metadata (31 lipid classes) from the package's identification module. Systematically enumerate fatty acyl chain combinations spanning 10–30 carbons (both even and odd) with 0–6 degrees of unsaturation for each of the 31 lipid classes, yielding 145,388 lipid species entries. For each species, compute theoretical m/z values using LiPydomics' documented ESI adduct lists (e.g., [M+H]+, [M+Na]+, [M-H]-) via the package's identification utilities. Populate predicted CCS values only for the 22 lipid classes having trained CCS models (scikit-learn regression on LipidCCS-style features such as fatty acyl chain length and unsaturation) and predicted HILIC retention times only for the 23 classes having trained RT models; write species, m/z/adduct, CCS, and RT predictions into SQLite/JSON tables matching the package's database schema. Rationale: machine-learning prediction fills gaps in experimental CCS collections, which do not yet comprehensively cover the vast lipid chemical space in terms of class and composition.

## Related tools

- **LiPydomics** (Source of lipid class definitions, identification-module adduct lists, and the target SQLite/JSON database schema) — https://github.com/dylanhross/lipydomics
- **Python** (Implementation language for the enumeration and database-writing script)
- **scikit-learn** (Provides the trained CCS and HILIC retention time regression models used to populate class-specific predictions)
- **SciPy** (Scientific computing support for the prediction/database workflow)
- **LipidCCS** (Feature scheme reference for the chain-length/unsaturation CCS prediction models)
- **LipidMass** (Source of lipid class building-block metadata used to define the enumeration space)

## Evaluation signals

- Database row count invariant: exactly 145,388 lipid species entries after enumeration
- CCS prediction coverage check: 94,451 predicted CCS values spanning exactly 22 lipid classes
- RT prediction coverage check: 106,020 predicted HILIC retention time values spanning exactly 23 lipid classes
- Schema check: species, m/z/adduct, CCS, and RT fields are populated per the package's database schema with CCS/RT left empty for classes lacking trained models
- Adduct correctness: each species carries theoretical m/z values computed from LiPydomics' documented ESI adduct lists

## Limitations

- CCS predictions exist only for the 22 lipid classes with trained models, and HILIC RT predictions only for the 23 classes with trained models — classes outside these sets cannot be populated
- Enumeration is bounded to fatty acyl chains of 10–30 carbons (even and odd) with 0–6 degrees of unsaturation; species outside this space are not covered
- Previous predicted CCS databases mostly cover mammalian lipid species with limited coverage of bacterial lipids; validation against species outside this design scope may be incomplete
- The package (including the lipid database and prediction models) is designed to be extensible as more data becomes available, so the enumerated totals reflect the models and class definitions at version 1.6.8

## Evidence

- [other] Systematic enumeration of fatty acyl chain combinations spanning 10–30 carbons with 0–6 degrees of unsaturation across 31 lipid classes yielding 145,388 species; CCS predictions for 22 classes and HILIC RT predictions for 23 classes were populated into SQLite/JSON database tables matching the package's schema.: [evidence span withheld — anchor status: not_found]
