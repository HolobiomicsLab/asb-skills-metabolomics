---
name: omsl-compound-identifier-parsing
description: Use when an OMSL file (.MSP, .MGF, .JSON, .CSV, .XML) contains inconsistent
  or missing compound identifiers (SMILES, InChI, InChIKey) and you must standardize
  them before dereplication, chemical-space plotting, or spectral deduplication.
when_to_use_negative:
- Input spectra already come from the FragHub output directory, where identifiers
  have been recalculated and normalized — re-parsing would duplicate work and risk
  introducing inconsistencies.
- Spectra lack both SMILES and InChI, precursor m/z, or adduct information — FragHub
  removes such inconsistent spectra during processing, so no valid identifier can
  be extracted.
- The goal is spectral (not chemical-identifier) deduplication — use SPLASH keys for
  spectral deduplication and reserve this skill for compound-identifier harmonization.
edam_operation: http://edamontology.org/operation_3431
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3473
tools:
- name: FragHub
  role: Workflow that parses the OMSLs, recalculates and normalizes SMILES/InChI/InChIKey
    via RDKit, and drops spectra lacking consistent identifiers
  repo: github.com/eMetaboHUB/FragHub
- name: RDKit
  role: Standardizes chemical identifiers and calculates exact and average molecular
    masses used to harmonize metadata across libraries
- name: PubChem
  role: Provides fingerprints and ontologic information resolved from the normalized
    InChIKey, used for downstream chemical-space projection
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:7a830e1510513c1bbc45e350a9dbfcb19335bec3dd36bce80bb82188e014e633
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  tools:
  - FragHub
  - PubChem
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_3431
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# omsl-compound-identifier-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

This skill parses compound chemical identifiers (SMILES, InChI, InChIKey) from the heterogeneous metadata of Open Mass Spectral Libraries such as GNPS, MoNA, MassBank, and MS-Dial-VS17, and recalculates/normalizes them with RDKit so downstream t-SNE chemical-space mapping, deduplication, and library harmonization are anchored on consistent identifiers.

## When to use

Use when an OMSL file (.MSP, .MGF, .JSON, .CSV, .XML) contains inconsistent or missing compound identifiers (SMILES, InChI, InChIKey) and you must standardize them before dereplication, chemical-space plotting, or spectral deduplication.

## When NOT to use

- Input spectra already come from the FragHub output directory, where identifiers have been recalculated and normalized — re-parsing would duplicate work and risk introducing inconsistencies.
- Spectra lack both SMILES and InChI, precursor m/z, or adduct information — FragHub removes such inconsistent spectra during processing, so no valid identifier can be extracted.
- The goal is spectral (not chemical-identifier) deduplication — use SPLASH keys for spectral deduplication and reserve this skill for compound-identifier harmonization.

## Inputs

- OMSL library files in .MSP, .MGF, .JSON, .CSV, or .XML format from GNPS, MoNA, MassBank, or MS-Dial-VS17
- Compound metadata fields carrying SMILES, InChI, or InChIKey strings
- Offline PubChem fingerprint data keyed by InChIKey
- NPClassifier API responses for superclassNP and PathwayNP coloration

## Outputs

- Standardized, recalculated chemical identifiers (SMILES, InChI, InChIKey) per compound
- Calculated exact and average molecular masses per compound
- Unique chemical identifier counts per library (e.g., 13,507 GNPS, 21,839 MoNA, 8,358 MassBank, 22,282 MS-Dial-VS17)
- Normalized InChIKey set usable as join keys for t-SNE fingerprint mapping or SPLASH-key deduplication

## How to apply

Parse the input library (e.g., GNPS .MGF, MoNA .MSP/.SDF/.JSON, MassBank, MS-Dial-VS17) to extract the compound metadata fields; spectra lacking SMILES and InChI, precursor m/z, or adduct information are dropped outright. Pass the extracted identifier strings through RDKit to standardize the chemical identifiers and calculate both exact and average molecular masses, yielding a uniform InChIKey per compound. Use the harmonized InChIKey as the join key across libraries (FragHub reduced 794,985 spectra from 35,673 unique identifiers to 602,744 spectra from 32,193 unique chemicals) and, for downstream chemical-space work, resolve each InChIKey to a PubChem fingerprint (offline PubChem data bundled with FragHub) and an NPClassifier superclass/pathway for coloration. The rationale is that each OMSL ships its own field conventions, so only recalculated identifiers from a single standardizer can be trusted as a merge key.

## Related tools

- **FragHub** (Workflow that parses the OMSLs, recalculates and normalizes SMILES/InChI/InChIKey via RDKit, and drops spectra lacking consistent identifiers) — github.com/eMetaboHUB/FragHub
- **RDKit** (Standardizes chemical identifiers and calculates exact and average molecular masses used to harmonize metadata across libraries)
- **PubChem** (Provides fingerprints and ontologic information resolved from the normalized InChIKey, used for downstream chemical-space projection)

## Examples

```
python -m fraghub --input gnps.mgf mona.msp massbank.xml msdial.csv --output fraghub_out/ --filters all
```

## Evaluation signals

- After processing, every retained spectrum carries recalculated SMILES, InChI, and InChIKey plus exact and average molecular masses derived from RDKit.
- Spectra without SMILES and InChI, precursor m/z, and adduct information are absent from the output — verify against the expected removal count.
- Identifier counts per library should approach the published values: 13,507 GNPS, 21,839 MoNA, 8,358 MassBank, 22,282 MS-Dial-VS17, 32,193 FragHub unique chemicals.
- When the normalized InChIKeys are joined to offline PubChem data, each compound resolves to a fingerprint block usable as the t-SNE molecular descriptor.
- Concatenated-pool unique identifier count drops from 35,673 to 32,193 chemicals after FragHub's harmonization, indicating successful identifier reconciliation.

## Limitations

- Dereplication via InChIKey does not capture collision-energy harmonization; the article notes harmonization of collision energies was not addressed due to ~70 different nonstandardized formats.
- Identifier-level merging alone does not resolve redundant spectra from overlapping libraries; SPLASH-key deduplication must complement it to avoid dot-product/reverse-dot-product scoring inconsistencies.
- MS-Dial-VS17 is the only presplit PI/negative library, so identifier extraction must handle already-split inputs differently from the other three OMSLs.

## Evidence
