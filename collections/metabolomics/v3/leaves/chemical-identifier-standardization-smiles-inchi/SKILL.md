---
name: chemical-identifier-standardization-smiles-inchi
description: Use when integrating heterogeneous OMSL entries (.MSP, .MGF, .JSON, .CSV)
  whose SMILES/InChI/InChIKey fields are inconsistent, missing, or nonstandard and
  must be normalized with RDKit before annotation workflows.
when_to_use_negative:
- Input records already carry valid, standardized SMILES/InChI/InChIKey and normalized
  masses, so re-derivation adds nothing but compute cost
- The goal is harmonizing collision energies — the article explicitly did not address
  them because measures occur in around 70 different nonstandardized formats
- The task is peak-list filtering (e.g. check_minimum_peak_requiered, normalize_intensity,
  reduce_peak_list) rather than chemical metadata normalization
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: RDKit
  role: Standardizes chemical identifiers (SMILES, InChI, InChIKey) and computes exact
    and average molecular masses during metadata cleaning
- name: PubChem
  role: Offline lookup used to complete ontologic information (names, descriptors)
    from InChIKeys
- name: FragHub
  role: Hosts the harmonization workflow, its mapping dictionaries, and the GUI that
    drives the processing stage
  repo: github.com/eMetaboHUB/FragHub
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:b5c31dbc469fffa9189a4053c68695d38975acc87a2f2ee4739b68c2700d7bba
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  tools:
  - Flash Entropy Search
  - FragHub
  - MS-Dial
  - PubChem
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chemical-identifier-standardization-smiles-inchi

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

FragHub's metadata-harmonization step for open mass spectral library (OMSL) entries: verify or re-derive SMILES, InChI and InChIKey with RDKit, recompute exact/average molecular masses, and complete PubChem-derived ontologic information so heterogeneous library records become interoperable.

## When to use

Use when integrating heterogeneous OMSL entries (.MSP, .MGF, .JSON, .CSV) whose SMILES/InChI/InChIKey fields are inconsistent, missing, or nonstandard and must be normalized with RDKit before annotation workflows.

## When NOT to use

- Input records already carry valid, standardized SMILES/InChI/InChIKey and normalized masses, so re-derivation adds nothing but compute cost
- The goal is harmonizing collision energies — the article explicitly did not address them because measures occur in around 70 different nonstandardized formats
- The task is peak-list filtering (e.g. check_minimum_peak_requiered, normalize_intensity, reduce_peak_list) rather than chemical metadata normalization

## Inputs

- OMSL library entries in .MSP, .MGF, .JSON, .CSV or .XML format (GNPS, MoNA, MassBank, MS-Dial-VS17, in-house MZmine3 output) carrying name/SMILES/InChI/InChIKey, adduct, precursor m/z and retention-time fields
- FragHub mapping dictionary of source-format field names to GNPS-compatible keys
- PubChem lookup data used offline to complete ontologies from InChIKeys

## Outputs

- Records with standardized, recalculated SMILES/InChI/InChIKey and RDKit-computed exact and average molecular masses
- Records with placeholder values (e.g. 'RT: 0.0', 'adduct: unknown') normalized to 'UNKNOWN' and tentative precursor m/z filled from exact mass plus adduct
- InChIKey-based PubChem ontology annotations and instrument/ionization descriptors normalized per HUPO PSI-MS
- Discarded-spectra records with removal reasons (spectra lacking both InChI and SMILES, or precursor m/z and adduct information)

## How to apply

Parse each library record's metadata fields (per the FragHub mapping dictionary for .MSP, .MGF, .JSON, .CSV, .XML keys) and translate them to GNPS-compatible naming. Use RDKit to verify or re-derive SMILES/InChI/InChIKey and to calculate both exact and average molecular masses; when the 'name' field is absent, replace it with the InChI. Map placeholder values such as 'RT: 0.0' or 'adduct: unknown' to 'UNKNOWN', update adduct/ion-mode/MS-level keys, and tentatively fill an empty precursor m/z from the exact mass and adduct. Complete ontologic information from PubChem databases on the InChIKey, mark the 'predicted' field, and normalize instrument model and ionization descriptors (QTOF, FT, ESI, APCI) with the HUPO PSI-MS hierarchical decision tree. Drop spectra lacking both InChI and SMILES — these are treated as inconsistent — and deduplicate on identical InChIKeys combined with identical fragment lists.

## Related tools

- **RDKit** (Standardizes chemical identifiers (SMILES, InChI, InChIKey) and computes exact and average molecular masses during metadata cleaning)
- **PubChem** (Offline lookup used to complete ontologic information (names, descriptors) from InChIKeys)
- **FragHub** (Hosts the harmonization workflow, its mapping dictionaries, and the GUI that drives the processing stage) — github.com/eMetaboHUB/FragHub

## Evaluation signals

- No output spectrum is missing SMILES, InChI, or a valid precursor m/z — the README states inconsistent spectra lacking these are removed
- InChIKey-based deduplication reduces the 794,985-spectrum / 35,673-identifier input to the reported 602,744 spectra / 32,193 unique chemical identifiers
- RDKit-recomputed exact and average molecular masses agree with the standardized structures and can fill tentative precursor m/z from exact mass plus adduct
- Placeholder values ('RT: 0.0', 'adduct: unknown') no longer appear; they are mapped to 'UNKNOWN'
- Field names/values match the GNPS-compatible mapping dictionary so outputs remain usable with MS-Dial, MZmine and Flash Entropy Search

## Limitations

- Spectra lacking both InChI and SMILES (or precursor m/z and adduct information) are removed entirely, shrinking compound coverage — 32,193 identifiers is low versus the estimated millions of natural products
- Collision energy harmonization was not addressed due to around 70 different nonstandardized measures
- No software version numbers are stated for RDKit, PubChem and related tools, complicating reproducibility of identifier re-derivation
- Chemical class disparity between OMSLs and natural product catalogues means this skill alone cannot fill annotation gaps; orthogonal strategies are needed

## Evidence

- [methods] RDKit is utilized to standardize chemical identifiers and calculate both exact and average molecular masses: "RDKit is utilized to standardize chemical identifiers and calculate both exact and average molecular masses"
- [readme] The program recalculates and normalizes chemical identifiers such as SMILES, InChI, and InChIKey, ensuring a uniform representation of molecular structures.: "The program recalculates and normalizes chemical identifiers such as SMILES, InChI, and InChIKey, ensuring a uniform representation of molecular structures."
- [readme] All spectra deemed inconsistent, i.e., those lacking SMILES and InChI, precursor m/z, and adduct information, are removed during the processing by FragHub.: "All spectra deemed inconsistent, i.e., those lacking SMILES and InChI, precursor m/z, and adduct information, are removed during the processing by FragHub."
- [readme] FragHub complete ontologic information from PubChem datas bases on Inchikey.: "FragHub complete ontologic information from PubChem datas bases on Inchikey."
- [intro] FragHub supports multiple OMSL formats (.MSP, .MGF, .JSON, .CSV, and .XML) and harmonizes metadata using RDKit15 and internal dictionaries.: "FragHub supports multiple OMSL formats (.MSP, .MGF, .JSON, .CSV, and .XML) and harmonizes metadata using RDKit15 and internal dictionaries."
