---
name: spectral-library-metadata-parsing
description: Use when integrating OMSLs such as GNPS, MoNA, MassBank, or MS-Dial-VS17
  and you need to extract harmonized InChIKeys per spectrum to quantify identifier
  overlap, per-library compound counts, and spectral redundancy.
when_to_use_negative:
- Input is already a single harmonized library in unified format — no cross-library
  parsing or identifier mapping is needed
- 'Spectra lack SMILES and InChI, precursor m/z, and adduct information: FragHub removes
  such inconsistent spectra, so overlap counts will be biased toward the refined set'
- The goal is collision-energy harmonization — the FragHub workflow explicitly does
  not address it because of around 70 different nonstandardized formats
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3520
tools:
- name: FragHub
  role: Workflow that parses OMSL files in .MSP/.MGF/.JSON/.CSV/.XML, standardizes
    field names/values, recalculates SMILES/InChI/InChIKey, and records SPLASH keys
    in update.json for deduplication
  repo: github.com/eMetaboHUB/FragHub
- name: Python
  role: Language used for parsing library files and tabulating identifier membership
    counts; FragHub is distributed as Python code
  repo: github.com/eMetaboHUB/FragHub
- name: spectra-hash (SPLASH)
  role: Generates database-independent spectral identifiers (SPLASH keys) used to
    deduplicate spectra during integration
  repo: github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:3500bd8ac54397fac51d7a5c8932010521462a261a5bb665e1e1214a5352940c
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
  - spectra-hash (SPLASH)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# spectral-library-metadata-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parses open mass spectral library files (.MSP, .MGF, .JSON, .CSV, .XML), extracts and normalizes per-compound chemical identifiers (SMILES, InChI, InChIKey) with RDKit, and builds an identifier-to-library membership map for cross-OMSL overlap analysis.

## When to use

Use when integrating OMSLs such as GNPS, MoNA, MassBank, or MS-Dial-VS17 and you need to extract harmonized InChIKeys per spectrum to quantify identifier overlap, per-library compound counts, and spectral redundancy.

## When NOT to use

- Input is already a single harmonized library in unified format — no cross-library parsing or identifier mapping is needed
- Spectra lack SMILES and InChI, precursor m/z, and adduct information: FragHub removes such inconsistent spectra, so overlap counts will be biased toward the refined set
- The goal is collision-energy harmonization — the FragHub workflow explicitly does not address it because of around 70 different nonstandardized formats

## Inputs

- GNPS library in .MGF format (13,507 compounds, 63,935 spectra)
- MoNA library in .MSP format (21,839 compounds, 190,359 spectra)
- MassBank records (8,358 compounds, 164,261 spectra)
- MS-Dial-VS17 library (22,282 compounds, 376,430 spectra)
- Generic .MSP, .MGF, .JSON, .CSV (semicolon- or tab-separated, peaks column), or .XML spectral library files

## Outputs

- InChIKey-to-library membership mapping over 35,673 unique chemical identifiers
- Per-library compound and spectrum counts
- Tabulation of identifiers shared by exactly 1, 2, 3, or 4 OMSLs
- Spectral duplicate counts per InChIKey within each library

## How to apply

Load each library file (GNPS .MGF, MoNA .MSP, MassBank records, MS-Dial-VS17) with a Python parser such as MatchMS or the FragHub workflow, whose initial step is parsing .MSP, .MGF, .JSON, .CSV, and .XML formats. For every spectrum, extract the chemical metadata and standardize identifiers (SMILES, InChI, InChIKey) with RDKit, which also calculates exact and average molecular masses. Count unique identifiers per source — expected values are 13,507 GNPS compounds, 21,839 MoNA, 8,358 MassBank, and 22,282 MS-Dial-VS17 — then build an InChIKey-to-library membership mapping across the 35,673 unique identifiers present in the integrated 794,985-spectra set. Tabulate how many identifiers appear in exactly one, two, three, or all four libraries, and count spectral duplicates per InChIKey within each library (e.g., MassBank's 164,261 spectra for 8,358 compounds) to quantify redundancy. The decision point is whether to report the raw integrated set or the FragHub-refined set (602,744 spectra / 32,193 unique chemicals), which drops spectra lacking SMILES/InChI, precursor m/z, or adduct information.

## Related tools

- **FragHub** (Workflow that parses OMSL files in .MSP/.MGF/.JSON/.CSV/.XML, standardizes field names/values, recalculates SMILES/InChI/InChIKey, and records SPLASH keys in update.json for deduplication) — github.com/eMetaboHUB/FragHub
- **Python** (Language used for parsing library files and tabulating identifier membership counts; FragHub is distributed as Python code) — github.com/eMetaboHUB/FragHub
- **spectra-hash (SPLASH)** (Generates database-independent spectral identifiers (SPLASH keys) used to deduplicate spectra during integration) — github.com/berlinguyinca/spectra-hash

## Examples

```
import pandas as pd; df = pd.read_csv('MS-Dial-VS17.csv', sep=';'); print(df['inchikey'].nunique())
```

## Evaluation signals

- Per-library compound counts must match published values: 13,507 (GNPS), 21,839 (MoNA), 8,358 (MassBank), 22,282 (MS-Dial-VS17)
- Integrated raw set must total 794,985 spectra for 35,673 unique chemical identifiers
- Fraction of identifiers shared between two or more OMSLs should reproduce the reported ~45% (19,419 compounds exclusive to a single library)
- All chemical identifiers must be standardized (RDKit-normalized SMILES, InChI, InChIKey) rather than taken verbatim from source files
- FragHub-refined output should shrink to 602,744 spectra / 32,193 unique chemicals after removing spectra lacking SMILES/InChI, precursor m/z, or adduct metadata

## Limitations

- FragHub removes all spectra deemed inconsistent (lacking SMILES and InChI, precursor m/z, and adduct information), so identifier totals differ between raw (35,673) and refined (32,193) sets
- Collision-energy harmonization is not performed due to around 70 nonstandardized formats, limiting metadata completeness of parsed records
- CSV inputs must be ';'- or tab-separated with quotechar '"' and a 'peaks' column; macOS and Linux support was removed in the 20_12_2024 release

## Evidence
