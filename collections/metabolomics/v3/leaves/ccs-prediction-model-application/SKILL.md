---
name: ccs-prediction-model-application
description: Use when an IM-MS lipidomics feature has an m/z and measured drift/collision
  cross section but no experimental CCS entry exists in the reference database, or
  when building a predicted lipid database covering fatty acyl chains of 10–30 carbons
  with 0–6 unsaturations.
when_to_use_negative:
- The lipid's CCS is already available in an experimental database — the experimental
  database of m/z and CCS covering 45 lipid classes should be used directly for higher-confidence
  identification.
- The target lipids are outside the 31 enumerated LipidMass-defined classes or fall
  outside the 10–30 carbon / 0–6 unsaturation enumeration space — the models will
  not cover them.
- The analysis goal is raw IM-MS data processing (peak picking, deconvolution) rather
  than lipid identification or database generation — LiPydomics assumes lipidomics
  data already in analyzable form, e.g. from MS-DIAL.
edam_operation: http://edamontology.org/operation_0321
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: LiPydomics
  role: Provides the trained scikit-learn CCS and HILIC RT prediction models, the
    LipidMass building-block class definitions, the ESI adduct m/z calculation utilities,
    and the database schema for storing predictions
  repo: https://github.com/dylanhross/lipydomics
- name: scikit-learn
  role: Underlying machine-learning framework for the trained CCS prediction models
    (22 lipid classes) and HILIC RT regression models (23 lipid classes)
- name: LipidCCS
  role: Reference for the LipidCCS-style feature scheme (fatty acyl chain length and
    unsaturation) used in the CCS prediction models
- name: LipidPioneer
  role: Cited prior work providing a user-generated exact mass template approach that
    the enumeration-based predicted database extends
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:4dbe2f9ef21c3f5d9dd15fd4a24fdcbbe3611de161ef25c029d0294df056e894
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
  - LipidPioneer
  - Python
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0321
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# ccs-prediction-model-application

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply LiPydomics' trained machine-learning CCS (and HILIC retention time) prediction models to populate a large predicted lipid database — 145,388 lipid species with 94,451 predicted CCS values (22 classes) and 106,020 HILIC RT values (23 classes) — enabling lipid identification when experimental CCS values are unavailable.

## When to use

Use when an IM-MS lipidomics feature has an m/z and measured drift/collision cross section but no experimental CCS entry exists in the reference database, or when building a predicted lipid database covering fatty acyl chains of 10–30 carbons with 0–6 unsaturations.

## When NOT to use

- The lipid's CCS is already available in an experimental database — the experimental database of m/z and CCS covering 45 lipid classes should be used directly for higher-confidence identification.
- The target lipids are outside the 31 enumerated LipidMass-defined classes or fall outside the 10–30 carbon / 0–6 unsaturation enumeration space — the models will not cover them.
- The analysis goal is raw IM-MS data processing (peak picking, deconvolution) rather than lipid identification or database generation — LiPydomics assumes lipidomics data already in analyzable form, e.g. from MS-DIAL.

## Inputs

- Lipid class definitions from the LiPydomics identification module (31 LipidMass-defined lipid classes with building-block metadata)
- Enumeration parameters: fatty acyl chain lengths (10–30 carbons, even and odd), degrees of unsaturation (0–6)
- Measured ion mobility collision cross section values from HILIC-IM-MS lipidomics data
- Measured HILIC retention times from HILIC-IM-MS lipidomics data
- Observed m/z values with ESI adduct annotations ([M+H]+, [M+Na]+, [M-H]-, etc.)

## Outputs

- Predicted lipid database of 145,388 lipid species entries in the package's SQLite/JSON database schema
- 94,451 predicted CCS values covering 22 lipid classes
- 106,020 predicted HILIC retention time values covering 23 lipid classes
- Lipid identifications at Level 3 (lipid class/subclass and fatty acid sum composition) for unknown features

## How to apply

Enumerate lipid species systematically using the LiPydomics/LipidMass building-block definitions (31 lipid classes; fatty acyl chain lengths 10–30 carbons, even and odd, with 0–6 degrees of unsaturation), yielding 145,388 species. For each species, compute theoretical m/z values from the package's documented ESI adduct lists ([M+H]+, [M+Na]+, [M-H]-, etc.) via the identification module. Query the trained scikit-learn CCS prediction models — which cover 22 lipid classes and use features such as fatty acyl chain length and unsaturation following the LipidCCS-style feature scheme — and the HILIC RT regression models covering 23 lipid classes. Write the species, m/z/adduct, CCS, and RT predictions into the package's SQLite/JSON database tables. During identification, match an unknown feature's measured CCS and RT against these predictions to assign a Level 3 identification (lipid class/subclass and fatty acid sum composition).

## Related tools

- **LiPydomics** (Provides the trained scikit-learn CCS and HILIC RT prediction models, the LipidMass building-block class definitions, the ESI adduct m/z calculation utilities, and the database schema for storing predictions) — https://github.com/dylanhross/lipydomics
- **scikit-learn** (Underlying machine-learning framework for the trained CCS prediction models (22 lipid classes) and HILIC RT regression models (23 lipid classes))
- **LipidCCS** (Reference for the LipidCCS-style feature scheme (fatty acyl chain length and unsaturation) used in the CCS prediction models)
- **LipidPioneer** (Cited prior work providing a user-generated exact mass template approach that the enumeration-based predicted database extends)

## Examples

```
from lipydomics.identification import annotate_dataset; annotate_dataset(dataset, level=3, tol=0.5)
```

## Evaluation signals

- Populated database contains exactly 145,388 lipid species entries, matching the number reported in the LiPydomics paper's abstract.
- Predicted CCS values total 94,451 and cover exactly 22 lipid classes (the classes having trained CCS models).
- Predicted HILIC retention time values total 106,020 and cover exactly 23 lipid classes (the classes having trained RT models).
- Theoretical m/z values for each species are consistent with the package's documented ESI adduct lists ([M+H]+, [M+Na]+, [M-H]-, etc.).
- Database tables conform to the package's SQLite/JSON database schema, and the database reproduces the paper's class coverage (45 lipid classes experimentally; 22 and 23 for CCS and RT predictions respectively).

## Limitations

- CCS prediction models cover only 22 of the 31 enumerated lipid classes and RT models only 23 — lipid classes without trained models cannot receive CCS or RT predictions.
- Previous predicted CCS databases (and by extension this approach's coverage motivation) mostly cover mammalian lipid species with limited coverage of bacterial lipids; coverage of the vast lipid chemical space is incomplete.
- The identification is at Level 3 (lipid class/subclass and fatty acid sum composition) — structural (sn-position, double-bond location) detail is not resolved.
- Predictions are ML-based estimates filling gaps in experimental CCS collections and are intended to be extensible as more experimental data becomes available.

## Evidence

- [abstract] Prediction models for CCS and HILIC retention time for 22 and 23 lipid classes, respectively, were trained: "Prediction models for CCS and HILIC retention time for 22 and 23 lipid classes, respectively, were trained"
- [abstract] enabled the generation of a large predicted lipid database with 145,388 entries: "enabled the generation of a large predicted lipid database with 145,388 entries"
- [intro] trained ML models for the prediction of CCS and HILIC retention times using simple but specialized feature sets, and built a predicted lipid database with a broad coverage of lipid classes: "trained ML models for the prediction of CCS and HILIC retention times using simple but specialized feature sets, and built a predicted lipid database with a broad coverage of lipid classes"
- [intro] this method generally allows Level 3 lipid identifications (lipid class/subclass and fatty acid sum composition): "this method generally allows Level 3 lipid identifications (lipid class/subclass and fatty acid sum composition)"
