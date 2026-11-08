---
name: hujo-psi-ms-ontology-normalization
description: Use when merging spectra from heterogeneous OMSLs (GNPS .MGF, MoNA .MSP/.SDF/.JSON,
  MassBank, MS-Dial) whose instrument models, ionization modes and RT units are inconsistent
  and must be standardized into a unified, downstream-searchable library.
when_to_use_negative:
- Input spectra already carry HUPO PSI-MS-controlled instrument and ionization annotations
  and consistent RT units — re-normalization adds nothing
- 'The goal is harmonizing collision energies: the article explicitly states collision-energy
  harmonization was not addressed because measures are varied and nonstandardized
  (around 70 different formats)'
- Spectra lack both InChI and SMILES, precursor m/z, and adduct information — these
  records are removed by FragHub rather than harmonized
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3318
tools:
- name: FragHub
  role: 'Runs the metadata harmonization: applies the mapping dictionary, the HUPO
    PSI-MS decision tree, the ''predicted'' flag, and RT normalization during library
    integration'
  repo: github.com/eMetaboHUB/FragHub
- name: RDKit
  role: Standardizes chemical identifiers (SMILES, InChI, InChIKey) and computes exact
    and average molecular masses, ensuring identifier consistency in the same harmonization
    pass
- name: PubChem
  role: Source of ontologic information completed by FragHub on InChIKey to enrich
    normalized records with chemical class descriptors
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:e95b9065124b53b393b396dc6fc86e9a3b4f3e3de8809d2a70f7d9a8a76c4470
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
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# hujo-psi-ms-ontology-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A metadata harmonization step from the FragHub OMSL integration workflow that maps free-text instrument model and ionization descriptors (QTOF, FT, ESI, APCI, etc.) onto an in-house HUPO PSI-MS hierarchical decision tree, marks the 'predicted' field, and normalizes retention times to minutes so that merged .MSP/.MGF/.JSON/.CSV libraries share consistent, searchable descriptors.

## When to use

Use when merging spectra from heterogeneous OMSLs (GNPS .MGF, MoNA .MSP/.SDF/.JSON, MassBank, MS-Dial) whose instrument models, ionization modes and RT units are inconsistent and must be standardized into a unified, downstream-searchable library.

## When NOT to use

- Input spectra already carry HUPO PSI-MS-controlled instrument and ionization annotations and consistent RT units — re-normalization adds nothing
- The goal is harmonizing collision energies: the article explicitly states collision-energy harmonization was not addressed because measures are varied and nonstandardized (around 70 different formats)
- Spectra lack both InChI and SMILES, precursor m/z, and adduct information — these records are removed by FragHub rather than harmonized

## Inputs

- Parsed spectra from OMSL files (.MSP, .MGF, .JSON, .CSV, .XML) with raw instrument model, ionization mode, adduct, adduct/ion-mode, RT, and predicted metadata fields
- In-house HUPO PSI-MS hierarchical decision tree (controlled-vocabulary mapping table)
- FragHub mapping dictionary translating source field names/values to GNPS-compatible naming

## Outputs

- Spectra records with HUPO PSI-MS-normalized instrument model and ionization descriptors (e.g. QTOF, FT, ESI, APCI)
- Populated 'predicted' field distinguishing experimental vs in-silico spectra
- Retention times normalized to minutes; placeholder values (e.g. 'RT: 0.0', 'adduct: unknown') replaced with 'UNKNOWN'
- Updated adduct/ion-mode/MS-level keys and tentatively filled precursor m/z (from exact mass and adduct), written into unified .MSP, .MGF, .JSON, .CSV library outputs

## How to apply

After parsing the source libraries (.MSP, .MGF, .JSON, .CSV, .XML) and mapping raw field names to GNPS-compatible keys with the FragHub mapping dictionary, pass each spectrum's instrument and ionization descriptors through the in-house HUPO PSI-MS hierarchical decision tree, which normalizes free-text values (e.g. QTOF, FT, ESI, APCI) into controlled-vocabulary terms. Explicitly set the 'predicted' field to distinguish experimental from in-silico spectra, converting placeholder values such as 'RT: 0.0' or 'adduct: unknown' to 'UNKNOWN'. Normalize retention times to minutes so that RT metadata is comparable across source libraries. Do this alongside RDKit-based cleaning (SMILES/InChI/InChIKey re-derivation, exact and average mass computation) and tentatively fill empty precursor m/z from exact mass and adduct before writing outputs, since MS-Dial, MZmine and Flash Entropy Search consume these normalized fields. Also normalize the PubChem-derived ontologic completion keyed on InChIKey so the split output partitions (positive/negative ionization, GC/LC, experimental/predicted) carry consistent descriptor values.

## Related tools

- **FragHub** (Runs the metadata harmonization: applies the mapping dictionary, the HUPO PSI-MS decision tree, the 'predicted' flag, and RT normalization during library integration) — github.com/eMetaboHUB/FragHub
- **RDKit** (Standardizes chemical identifiers (SMILES, InChI, InChIKey) and computes exact and average molecular masses, ensuring identifier consistency in the same harmonization pass)
- **PubChem** (Source of ontologic information completed by FragHub on InChIKey to enrich normalized records with chemical class descriptors)

## Evaluation signals

- Every retained output spectrum carries valid SMILES, InChI, and a valid precursor m/z — no output spectrum is missing these fields
- Instrument/ionization values in the integrated library resolve to HUPO PSI-MS hierarchical terms (QTOF, FT, ESI, APCI, etc.) with no leftover source-specific free text
- All retention times are expressed in minutes and placeholder values ('RT: 0.0', 'adduct: unknown') appear as 'UNKNOWN'
- The 'predicted' field is populated for every spectrum and the experimental-vs-predicted output partitions are mutually consistent with it
- Integrated library totals match the reported refinement: 602,744 spectra across 32,193 unique chemical identifiers from the 794,985-spectrum / 35,673-identifier input, with outputs usable by MS-Dial, MZmine and Flash Entropy Search

## Limitations

- Collision energies could not be harmonized because their measures are varied and nonstandardized (around 70 different formats)
- No software version numbers are stated for tools involved (e.g., RDKit, PubChem), affecting reproducibility of the normalization
- In recent FragHub releases, MacOS and Linux support was removed and the license changed from MIT to CC-BY-NC 4.0, imposing non-commercial restrictions
- Chemical class coverage of the harmonized library (32,193 compounds) is low relative to the estimated several million natural products, requiring orthogonal strategies to fill the gap

## Evidence

- [methods] Normalize instrument model and ionization descriptors (QTOF, FT, ESI, APCI, etc.) using the in-house HUPO PSI-MS hierarchical decision tree: [evidence span withheld — anchor status: not_found]
