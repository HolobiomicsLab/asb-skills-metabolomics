---
name: mass-spectra-metadata-harmonization
description: Use when concatenating OMSLs from GNPS (.MGF), MoNA (.MSP/.SDF/.JSON),
  MassBank, or MS-Dial-VS17 into a single library and you need to harmonize metadata
  fields (SMILES, InChI, precursor m/z, adduct, ion mode, instrument type) so the
  output is usable with MS-Dial, MZmine, and Flash Entropy Search.
when_to_use_negative:
- Spectra already in a single, harmonized library format — running harmonization again
  risks stripping valid metadata through the 'UNKNOWN' fallback mapping and re-dropping
  spectra.
- 'Collision energy harmonization is required: the article explicitly did not address
  it because ~70 different nonstandardized energy formats exist across OMSLs.'
- Input contains only peak lists without any compound identifier (SMILES or InChI)
  and without adduct or precursor m/z — these spectra are removed during processing.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FragHub
  role: Primary workflow that parses OMSLs, harmonizes metadata via RDKit and internal
    dictionaries, applies SPLASH-key deduplication, and partitions spectra
  repo: https://github.com/eMetaboHUB/FragHub
- name: RDKit
  role: Standardizes chemical identifiers (SMILES, InChI, InChIKey) and computes exact
    and average molecular masses
- name: spectra-hash
  role: Generates SPLASH spectral hash identifiers for deduplication across library
    updates
  repo: https://github.com/berlinguyinca/spectra-hash
- name: MZmine
  role: Source of in-house spectral library exports in .JSON, .MGF, and .MSP formats
- name: MS-Dial
  role: Target analysis software that consumes the unified .MSP output
- name: Flash Entropy Search
  role: Target spectral-search engine that consumes the unified output
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:234be7fe66b5a9e7fba12cdd6ae8905141e2e2c3a6971e77f677fe8ac3e254e2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  repo_url: https://github.com/eMetaboHUB/FragHub
  tools:
  - Flash Entropy Search
  - FragHub
  - MS-Dial
  - MZmine
  - Python
  - RDKit
  - spectra-hash
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# mass-spectra-metadata-harmonization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes heterogeneous open mass spectral library (OMSL) metadata — field names, chemical identifiers, ionization modes, retention times, instrument descriptors — into a unified GNPS-compatible format, as implemented in the FragHub workflow (Analytical Chemistry 2024).

## When to use

Use when concatenating OMSLs from GNPS (.MGF), MoNA (.MSP/.SDF/.JSON), MassBank, or MS-Dial-VS17 into a single library and you need to harmonize metadata fields (SMILES, InChI, precursor m/z, adduct, ion mode, instrument type) so the output is usable with MS-Dial, MZmine, and Flash Entropy Search.

## When NOT to use

- Spectra already in a single, harmonized library format — running harmonization again risks stripping valid metadata through the 'UNKNOWN' fallback mapping and re-dropping spectra.
- Collision energy harmonization is required: the article explicitly did not address it because ~70 different nonstandardized energy formats exist across OMSLs.
- Input contains only peak lists without any compound identifier (SMILES or InChI) and without adduct or precursor m/z — these spectra are removed during processing.

## Inputs

- Open mass spectral library files: GNPS .MGF, MoNA .MSP/.SDF/.JSON, MassBank records, MS-Dial-VS17 exports
- In-house MZmine3 spectral library generation module output
- CSV peak tables with ';'- or tab-separated columns and a 'peaks' column quoted with '"'
- GNPS-compatible metadata mapping dictionaries (bundled with FragHub)
- HUPO PSI-MS controlled-vocabulary hierarchy for instrument model and ionization descriptors

## Outputs

- Unified, harmonized spectral library files in .MSP, .MGF, .JSON, and .CSV formats
- Spectra partitioned by ionization mode (positive/negative), chromatographic mode (GC/LC), and acquisition type (experimental/in silico)
- update.json file recording SPLASH keys of all processed spectra for incremental deduplication
- Log of discarded spectra with removal reasons (missing SMILES/InChI, missing precursor m/z, failed filters)

## How to apply

Parse the source .MSP, .MGF, .JSON, .CSV, or .XML files and use FragHub's mapping dictionary to translate source-specific keys into GNPS-compatible naming, keeping the output interoperable with downstream annotation tools. Generate a SPLASH key for every spectrum using the spectra-hash algorithm (Wohlgemuth et al.), record all keys in 'update.json', and skip spectra whose SPLASH key is already present so only new entries are integrated on update. Clean and normalize metadata with RDKit: standardize or re-derive SMILES/InChI/InChIKey, drop spectra lacking both InChI and SMILES, compute exact and average molecular masses, replace an absent 'name' with the InChI, map placeholders like 'RT: 0.0' or 'adduct: unknown' to 'UNKNOWN', and tentatively fill empty precursor m/z from exact mass and adduct. Normalize instrument model and ionization descriptors (QTOF, FT, ESI, APCI) against a HUPO PSI-MS hierarchical decision tree, convert retention times to minutes, and mark spectra as predicted vs. experimental. Split the harmonized spectra into partitions by ionization mode (positive/negative), separation technique (GC/LC), and experimental/in silico origin, removing duplicates with identical InChIKeys and fragment lists, then write unified outputs in .MSP, .MGF, .JSON, and .CSV formats.

## Related tools

- **FragHub** (Primary workflow that parses OMSLs, harmonizes metadata via RDKit and internal dictionaries, applies SPLASH-key deduplication, and partitions spectra) — https://github.com/eMetaboHUB/FragHub
- **RDKit** (Standardizes chemical identifiers (SMILES, InChI, InChIKey) and computes exact and average molecular masses)
- **spectra-hash** (Generates SPLASH spectral hash identifiers for deduplication across library updates) — https://github.com/berlinguyinca/spectra-hash
- **MZmine** (Source of in-house spectral library exports in .JSON, .MGF, and .MSP formats)
- **MS-Dial** (Target analysis software that consumes the unified .MSP output)
- **Flash Entropy Search** (Target spectral-search engine that consumes the unified output)

## Examples

```
python -m fraghub --input GNPS.mgf MoNA.msp MassBank.json MS-Dial-VS17.msp --output out_dir --formats msp mgf json csv --filters check_minimum_peak_requiered,remove_peak_above_precursormz
```

## Evaluation signals

- The integrated library contains 602,744 spectra across 32,193 unique chemical identifiers when integrating GNPS, MoNA, MassBank, and MS-Dial-VS17 (from 794,985 spectra / 35,673 identifiers input).
- No output spectrum lacks SMILES, InChI, or a valid precursor m/z — inconsistent spectra missing these are removed by FragHub.
- Every output spectrum carries a valid SPLASH key of the form splash10-xxxx-xxxxxxxxxx-<hash> (e.g. splash10-0002-0900000000-b112e4e059e1ecf98c5f), and update.json contains one key per retained spectrum.
- Metadata fields map to GNPS-compatible keys so the output loads directly in MS-Dial, MZmine, and Flash Entropy Search without manual re-editing.
- Instrument and ionization descriptors conform to the HUPO PSI-MS hierarchy (QTOF, FT, ESI, APCI), and retention times are expressed in minutes.

## Limitations

- Collision energies are not harmonized (~70 different nonstandardized formats across sources).
- Coverage of ~32,000 unique compounds is low relative to the estimated several-million natural product diversity; orthogonal strategies are needed to fill the gap.
- Spectra lacking both SMILES and InChI, precursor m/z, or adduct information are discarded outright.
- Recent releases removed MacOS and Linux support and changed the license from MIT to CC-BY-NC 4.0, imposing non-commercial restrictions.
- Placeholders such as 'RT: 0.0' or 'adduct: unknown' are silently replaced with 'UNKNOWN', which may hide incomplete source metadata.

## Evidence

- [methods] SPLASH keys are recorded in update.json so only new spectra are processed.: [evidence span withheld — anchor status: not_found]
