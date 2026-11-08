---
name: lipid-species-enumeration-fatty-acyl-chain-space
description: Use when a lipidomics identification module needs a comprehensive predicted
  lipid database spanning class and composition space beyond available experimental
  CCS/RT collections, especially for HILIC-IM-MS workflows.
when_to_use_negative:
- The analysis only needs identifications against experimentally measured values —
  the experimental database of m/z and CCS of 45 lipid classes (23 with HILIC retention
  times) already covers the query, and predicted entries would be unnecessary.
- The lipid space of interest falls outside the enumeration grid — e.g., acyl chains
  shorter than 10 or longer than 30 carbons, more than 6 degrees of unsaturation,
  or lipid classes not defined in the LipidMass-based 31-class set.
- The input is already an observed feature list (m/z, CCS, RT) requiring lipid identification
  at different confidence levels rather than generation of a theoretical species database.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3318
tools:
- name: LiPydomics
  role: Provides the lipid class definitions, identification-module building-block
    metadata, ESI adduct lists, and database schema into which enumerated species
    and predictions are written
  repo: https://github.com/dylanhross/lipydomics
- name: scikit-learn
  role: Hosts the trained machine-learning regression models used to predict CCS (22
    classes) and HILIC retention times (23 classes) for enumerated species
- name: LipidCCS
  role: Source of the specialized feature scheme (fatty acyl chain length and unsaturation
    features) on which the CCS prediction models are built
- name: SciPy
  role: Scientific-computing foundation used alongside Python for the enumeration
    and model work
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:3e59b9cc8c403ea547746684ca183e8683149096b4bedbcd728420d3920669d7
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
  - Python
  - SciPy
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lipid-species-enumeration-fatty-acyl-chain-space

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematically enumerates lipid species for LipidMass-defined lipid classes by combining fatty acyl chain lengths (10–30 carbons, even and odd) and degrees of unsaturation (0–6), then attaches theoretical m/z values (ESI adducts such as [M+H]+, [M+Na]+, [M-H]-), predicted CCS values (22 classes) and predicted HILIC retention times (23 classes) to build a broad predicted lipid database of 145,388 entries.

## When to use

Use when a lipidomics identification module needs a comprehensive predicted lipid database spanning class and composition space beyond available experimental CCS/RT collections, especially for HILIC-IM-MS workflows.

## When NOT to use

- The analysis only needs identifications against experimentally measured values — the experimental database of m/z and CCS of 45 lipid classes (23 with HILIC retention times) already covers the query, and predicted entries would be unnecessary.
- The lipid space of interest falls outside the enumeration grid — e.g., acyl chains shorter than 10 or longer than 30 carbons, more than 6 degrees of unsaturation, or lipid classes not defined in the LipidMass-based 31-class set.
- The input is already an observed feature list (m/z, CCS, RT) requiring lipid identification at different confidence levels rather than generation of a theoretical species database.

## Inputs

- LiPydomics lipid class definitions (31 LipidMass-based lipid classes with building-block metadata from the identification module)
- LiPydomics documented ESI adduct lists for theoretical m/z calculation
- Trained CCS prediction models (22 lipid classes, scikit-learn, LipidCCS-style features)
- Trained HILIC retention time prediction models (23 lipid classes, scikit-learn regression on class-specific features)

## Outputs

- Predicted lipid database of 145,388 lipid species entries (SQLite/JSON tables matching the LiPydomics database schema)
- 94,451 predicted CCS values covering 22 lipid classes
- 106,020 predicted HILIC retention time values covering 23 lipid classes
- Per-species theoretical m/z values per ESI adduct

## How to apply

Load the LiPydomics lipid class definitions and LipidMass-based building-block metadata (31 lipid classes) from the package's identification module. Systematically enumerate fatty acyl chain combinations spanning 10–30 carbons (both even and odd chain lengths) with 0–6 degrees of unsaturation for each lipid class, yielding 145,388 lipid species entries. For each enumerated species, compute theoretical m/z values using LiPydomics' documented ESI adduct lists (e.g., [M+H]+, [M+Na]+, [M-H]-). Populate predicted CCS values only for the 22 lipid classes having trained CCS models, and predicted HILIC retention times only for the 23 classes having trained RT models — both models are scikit-learn regressions built on specialized feature sets (fatty acyl chain length and unsaturation features, following the LipidCCS-style feature scheme). Write the species, m/z/adduct, CCS, and RT predictions into the package's database tables (SQLite/JSON schema). The rationale is that experimental CCS collections do not yet comprehensively cover the vast lipid chemical space, so ML-predicted CCS and RT values fill the gaps with class-specific predictions.

## Related tools

- **LiPydomics** (Provides the lipid class definitions, identification-module building-block metadata, ESI adduct lists, and database schema into which enumerated species and predictions are written) — https://github.com/dylanhross/lipydomics
- **scikit-learn** (Hosts the trained machine-learning regression models used to predict CCS (22 classes) and HILIC retention times (23 classes) for enumerated species)
- **LipidCCS** (Source of the specialized feature scheme (fatty acyl chain length and unsaturation features) on which the CCS prediction models are built)
- **SciPy** (Scientific-computing foundation used alongside Python for the enumeration and model work)

## Evaluation signals

- The populated database reproduces the reported total of 145,388 lipid species entries.
- CCS predictions are present for exactly 22 lipid classes (94,451 predicted CCS values).
- HILIC retention time predictions are present for exactly 23 lipid classes (106,020 predicted RT values).
- Enumeration grid check: every species combines acyl chain lengths of 10–30 carbons (even and odd) with 0–6 degrees of unsaturation across the 31 LipidMass-defined classes.
- Schema check: database tables carry species identity, theoretical m/z per adduct, CCS, and RT fields consistent with the package's database schema.

## Limitations

- Coverage is bounded by the enumeration grid: 10–30 carbons (even and odd) and 0–6 unsaturations for LipidMass-defined classes — species outside this space are absent.
- Predicted CCS values exist only for the 22 lipid classes with trained models, and HILIC RT values only for the 23 classes with trained models; classes lacking models carry no predictions.
- Previous/predicted databases of this kind mostly cover mammalian lipid species and have limited coverage of bacterial lipids; the package (including the lipid database and prediction models) is extensible, allowing expansion as more data becomes available.

## Evidence
