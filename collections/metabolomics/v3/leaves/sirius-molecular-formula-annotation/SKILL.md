---
name: sirius-molecular-formula-annotation
description: Use when you need a baseline formula/structure annotation rate for MS/MS
  query spectra (e.g., GMN anchor–suspect pairs) against a defined structure database
  such as bioDB.
when_to_use_negative:
- Your goal is to generate and rank novel biotransformation derivatives for unknown
  suspects — that is the BAM/PROXIMAL2/GNN-SOM pipeline, not the SIRIUS baseline
- Spectra lack associated GNPS/MassIVE metadata (adduct, ionization mode, instrument
  type), which SIRIUS configuration here depends on
- You need annotation against a database other than the biomolecule database (bioDB)
  used to define this data set
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SIRIUS 5.6.3
  role: Command-line molecular formula annotation followed by molecular structure
    annotation against bioDB on each query spectrum
merged_aliases:
- molecular-formula-annotation-evaluation
- sirius-molecular-formula-prediction
merged_alias_records:
- alias: sirius-molecular-structure-annotation
  slug: sirius-molecular-structure-annotation
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:0f4c3e3191e4082613c985a13fe8ae68d90bc6269bc3800d2e25c2dfdcb687c9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSNovelist
  - SIRIUS
  - SIRIUS 5.6.3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# sirius-molecular-formula-annotation

## Summary

Run SIRIUS 5.6.3 molecular formula (and conditional structure) annotation on MS/MS query spectra to establish a structural elucidation baseline for the BAM anchor–suspect evaluation.

## When to use

Use when you need a baseline formula/structure annotation rate for MS/MS query spectra (e.g., GMN anchor–suspect pairs) against a defined structure database such as bioDB.

## When NOT to use

- Your goal is to generate and rank novel biotransformation derivatives for unknown suspects — that is the BAM/PROXIMAL2/GNN-SOM pipeline, not the SIRIUS baseline
- Spectra lack associated GNPS/MassIVE metadata (adduct, ionization mode, instrument type), which SIRIUS configuration here depends on
- You need annotation against a database other than the biomolecule database (bioDB) used to define this data set

## Inputs

- 38,368 unique MS/MS query spectra from the global molecular network anchor–suspect data set (filtered for compounds present in bioDB)
- GNPS/MassIVE metadata per spectrum (adduct, ionization mode, instrument type)
- bioDB structural database

## Outputs

- Per-query SIRIUS molecular formula annotations
- Per-query SIRIUS molecular structure annotations (only for spectra with generated formulas)
- Prediction table with one row per query (38,368 rows) and counts of correct formulas (4756) and correct structures (4560)

## How to apply

Load the unique query spectra (here, the 38,368 anchor–suspect spectra filtered for compounds present in bioDB, derived from the global molecular network) and configure SIRIUS 5.6.3 via the CLI, taking the adduct, ionization mode, and instrument type from the corresponding GNPS/MassIVE metadata. Set the default element set (C, H, N, O, P, S, Br, Cl, B, Se) and 10 ppm mass accuracy, and run molecular formula annotation against the biomolecule structure database (bioDB) for each query spectrum. Follow the paper's two-stage decision point: run molecular structure annotation against bioDB only for spectra where the correct molecular formula was generated. Aggregate per-query results into a prediction table and count correctly annotated formulas and structures (4756 and 4560, respectively) as the baseline performance figures.

## Related tools

- **SIRIUS 5.6.3** (Command-line molecular formula annotation followed by molecular structure annotation against bioDB on each query spectrum)

## Examples

```
sirius analyze --input queries.mgf --database bioDB --elements CHNOPSBrClBSe --ppm-max 10
```

## Evaluation signals

- Prediction table row count equals 38,368 (one row per query spectrum)
- Reported true molecular formula count equals 4756
- Reported true molecular structure count equals 4560
- Structure annotation was run only for spectra where a molecular formula was generated (two-stage gating)

## Limitations

- SIRIUS performance is measured only against bioDB entries; queries outside bioDB are excluded from the data set
- Molecular structure annotation is gated on formula generation, so structure counts cannot exceed formula counts
- Configuration (adduct, ionization mode, instrument type) is inherited from GNPS/MassIVE metadata, so metadata errors propagate

## Evidence

- [methods] SIRIUS formula-then-structure gating: "we first performed molecular formula annotation, and then only if the correct formula was generated, we performed molecular structure annotation."
- [methods] SIRIUS version and data set scope: [evidence span withheld — anchor status: not_found]
