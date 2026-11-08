---
name: theoretical-mz-computation-esi-adduct-lists
description: Use when building or extending a predicted lipid database and monoisotopic
  m/z values for ESI adducts are needed to match IM-MS features against enumerated
  lipid species.
when_to_use_negative:
- When only experimental (not predicted) m/z and CCS reference values are needed —
  LiPydomics also assembles a comprehensive experimental database of m/z and CCS of
  45 lipid classes, a distinct resource
- When identifying unknown non-lipid metabolites or lipid species outside the 10-30
  carbon / 0-6 unsaturation enumeration space
- When lipid identification requires MS/MS-level evidence beyond the Level 3 identifications
  (lipid class/subclass and fatty acid sum composition) supported by HILIC-IM-MS
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3318
tools:
- name: LiPydomics
  role: Provides the identification module and documented ESI adduct lists used to
    compute theoretical m/z values and store predicted database entries
  repo: https://github.com/dylanhross/lipydomics
- name: Python
  role: Runtime language in which the enumerated species, adduct m/z computations,
    and SQLite/JSON database writing are implemented
- name: scikit-learn
  role: Trains the CCS (22 lipid classes) and HILIC RT (23 lipid classes) prediction
    models whose outputs are attached to the computed m/z entries
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:610deddabf17fd92a0b931b6a84c2642df9f0f962a24e77bfbe0fc7ddfd3b5fa
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# theoretical-mz-computation-esi-adduct-lists

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for computing theoretical m/z values for enumerated lipid species using LiPydomics' documented ESI adduct lists ([M+H]+, [M+Na]+, [M-H]-, etc.), producing the predicted lipid database entries that support Level 3 lipid identification in HILIC-IM-MS lipidomics.

## When to use

Use when building or extending a predicted lipid database and monoisotopic m/z values for ESI adducts are needed to match IM-MS features against enumerated lipid species.

## When NOT to use

- When only experimental (not predicted) m/z and CCS reference values are needed — LiPydomics also assembles a comprehensive experimental database of m/z and CCS of 45 lipid classes, a distinct resource
- When identifying unknown non-lipid metabolites or lipid species outside the 10-30 carbon / 0-6 unsaturation enumeration space
- When lipid identification requires MS/MS-level evidence beyond the Level 3 identifications (lipid class/subclass and fatty acid sum composition) supported by HILIC-IM-MS

## Inputs

- LiPydomics lipid class definitions and LipidMass building-block metadata (31 lipid classes)
- Enumeration ranges: fatty acyl chain lengths 10-30 carbons (even and odd), 0-6 degrees of unsaturation
- LiPydomics documented ESI adduct lists ([M+H]+, [M+Na]+, [M-H]-, etc.)

## Outputs

- Predicted lipid database with 145,388 lipid species entries (SQLite/JSON tables matching the lipydomics schema)
- Theoretical m/z values per species for each ESI adduct
- Predicted CCS values for 22 lipid classes (94,451 values)
- Predicted HILIC retention times for 23 lipid classes (106,020 values)

## How to apply

Load the LiPydomics lipid class definitions and LipidMass-based building-block metadata (31 lipid classes) from the package's identification module, then systematically enumerate fatty acyl chain combinations spanning 10-30 carbons (even and odd) with 0-6 degrees of unsaturation per class, yielding 145,388 lipid species entries. For each enumerated species, compute theoretical m/z values using LiPydomics' documented ESI adduct lists (e.g., [M+H]+, [M+Na]+, [M-H]-, etc.) via the package's identification utilities. Optionally pair each m/z/adduct entry with predicted CCS values (94,451 values across 22 lipid classes, scikit-learn models following the LipidCCS-style feature scheme) and predicted HILIC retention times (106,020 values across 23 lipid classes). Write species, m/z/adduct, CCS, and RT predictions into SQLite/JSON tables matching the package's database schema. Validate by checking that the populated database reproduces 145,388 lipid species entries with the reported class coverages.

## Related tools

- **LiPydomics** (Provides the identification module and documented ESI adduct lists used to compute theoretical m/z values and store predicted database entries) — https://github.com/dylanhross/lipydomics
- **Python** (Runtime language in which the enumerated species, adduct m/z computations, and SQLite/JSON database writing are implemented)
- **scikit-learn** (Trains the CCS (22 lipid classes) and HILIC RT (23 lipid classes) prediction models whose outputs are attached to the computed m/z entries)

## Evaluation signals

- Total species count in the populated database equals the reported 145,388 entries
- CCS predictions cover exactly 22 lipid classes totaling 94,451 values; HILIC RT predictions cover exactly 23 lipid classes totaling 106,020 values
- Each species' computed m/z matches the theoretical monoisotopic mass plus the specified adduct's mass shift for its charge state
- Database tables conform to the package's SQLite/JSON schema (species, m/z/adduct, CCS, RT fields populated)
- Enumeration reproduces the stated grid: 10-30 carbons (even and odd) x 0-6 unsaturations across the 31 LipidMass-defined lipid classes

## Limitations

- Coverage is bounded by the enumeration grid (10-30 carbons, 0-6 unsaturations) and the 31 LipidMass-defined lipid classes; species outside this space are absent
- Adduct lists are documented but extensible — the package is 'built to be highly extensible and customizable, allowing easy expansion as more data becomes available', so adduct or class coverage may change with package versions
- Prior predicted CCS databases covered mostly mammalian lipids with limited bacterial coverage; users matching bacterial lipids (e.g., the Staphylococcus aureus demonstration) should verify class coverage of the predicted models

## Evidence

- [other] compute theoretical m/z values using LiPydomics' documented ESI adduct lists (e.g., [M+H]+, [M+Na]+, [M-H]-, etc.): [evidence span withheld — anchor status: not_found]
