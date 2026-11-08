---
name: multi-source-chemical-set-intersection
description: Use when several spectral libraries (e.g., .MGF, .MSP files from GNPS,
  MoNA, MassBank, MS-Dial) have been harmonized and you need to know what fraction
  of unique compounds is shared vs. exclusive to each source.
when_to_use_negative:
- Input spectra lack recalcuable SMILES/InChI and precursor/adduct metadata, so no
  reliable InChIKey can be derived for set membership
- You need collision-energy or peak-level spectral deduplication rather than compound-level
  identifier overlap (SPLASH keys, not InChIKeys, are the appropriate unit there)
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: FragHub
  role: Integrates the OMSLs into a unified format and harmonizes chemical identifiers
    (InChIKey) so per-source identifier sets are comparable
  repo: github.com/eMetaboHUB/FragHub
- name: RDKit
  role: Standardizes SMILES/InChI/InChIKey chemical identifiers used as the set-intersection
    key
- name: Python
  role: Implements the set-membership mapping and intersection counting over parsed
    library files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:5a3778628332d972297a104f8acdd5a23d2ddd4dc9ab71f2bab647dbeff9e0a0
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
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# multi-source-chemical-set-intersection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies overlap of unique chemical identifiers (harmonized InChIKeys) across multiple open mass spectral libraries (OMSLs) by building an identifier-to-library membership mapping and tabulating identifiers found in exactly one, two, three, or all sources.

## When to use

Use when several spectral libraries (e.g., .MGF, .MSP files from GNPS, MoNA, MassBank, MS-Dial) have been harmonized and you need to know what fraction of unique compounds is shared vs. exclusive to each source.

## When NOT to use

- Input spectra lack recalcuable SMILES/InChI and precursor/adduct metadata, so no reliable InChIKey can be derived for set membership
- You need collision-energy or peak-level spectral deduplication rather than compound-level identifier overlap (SPLASH keys, not InChIKeys, are the appropriate unit there)

## Inputs

- Processed OMSL library files: GNPS .MGF, MoNA .MSP, MassBank records, MS-Dial-VS17 library
- Harmonized InChIKey (plus SMILES/InChI) per compound, as standardized by FragHub/RDKit
- Per-library spectrum-to-compound mappings (e.g., 794,985 spectra total)

## Outputs

- InChIKey-to-library membership table over 35,673 unique chemical identifiers
- Counts of identifiers appearing in exactly one, two, three, or all four libraries
- Shared-identifier fraction (~45%) and per-library unique compound and spectrum counts
- Spectral redundancy counts per InChIKey within each library

## How to apply

Parse each library file (GNPS .MGF, MoNA .MSP, MassBank, MS-Dial-VS17) and extract the FragHub-harmonized InChIKey for every compound, expecting per-library unique counts such as 13,507 (GNPS), 21,839 (MoNA), 8,358 (MassBank), and 22,282 (MS-Dial-VS17). Build an InChIKey-to-library membership map over the integrated set — here 35,673 unique chemical identifiers across 794,985 spectra — and count identifiers appearing in exactly one, two, three, or all four libraries. Optionally count spectral duplicates per InChIKey within each library (e.g., MassBank's 164,261 spectra for 8,358 compounds) to quantify redundancy that may affect dot-product/reverse-dot-product scoring. Report the shared fraction; validation requires reproducing the paper's result that approximately 45% of the 35,673 unique identifiers are shared between two or more OMSLs, with 19,419 compounds exclusive to a single library. Identifier harmonization should rely on RDKit-standardized SMILES, InChI, and InChIKey values rather than raw library annotations, since inconsistent fields are a key reason identifiers disagree across sources.

## Related tools

- **FragHub** (Integrates the OMSLs into a unified format and harmonizes chemical identifiers (InChIKey) so per-source identifier sets are comparable) — github.com/eMetaboHUB/FragHub
- **RDKit** (Standardizes SMILES/InChI/InChIKey chemical identifiers used as the set-intersection key)
- **Python** (Implements the set-membership mapping and intersection counting over parsed library files)

## Examples

```
shared = set(gnps_keys) & set(mona_keys) & set(massbank_keys) & set(msdial_keys); print(len(shared), len(shared)/35673)
```

## Evaluation signals

- Per-library unique compound counts must match the reported values: 13,507 GNPS, 21,839 MoNA, 8,358 MassBank, 22,282 MS-Dial-VS17
- Union of all per-source identifier sets must equal 35,673 unique chemical identifiers over 794,985 spectra
- Shared fraction (identifiers in ≥2 libraries) must reproduce ~45%, with 19,419 single-library-exclusive compounds
- Every InChIKey in the mapping must be RDKit-standardized; spectra lacking SMILES/InChI, precursor m/z, and adduct information should have been excluded upstream
- Sum of the four per-library spectrum counts (63,935 + 190,359 + 164,261 + 376,430) must reconcile with the 794,985 total

## Limitations

- Redundancy across OMSLs (multiple spectra per InChIKey) can cause inconsistencies with dot product and reverse dot product scoring, which are sensitive to fragment number and intensity
- Post-refinement integration retains only 32,193 unique chemicals after FragHub filtering (602,744 spectra), which changes the denominator relative to the raw 35,673-identifier set
- Identifier overlap conflates structural identity with spectral coverage: two libraries may share an InChIKey but contain non-redundant spectra (different collision energies, which FragHub does not harmonize)

## Evidence
