---
name: spectral-metadata-normalization-rdkit
description: Use when integrating or standardizing spectra from heterogeneous OMSLs
  (GNPS .MGF, MoNA .MSP/.SDF/.JSON, MassBank, MS-Dial) so that spectra carry consistent
  identifiers, masses, and ontology terms for downstream annotation tools.
when_to_use_negative:
- Input is a single self-consistent library already using standardized fields and
  identifiers, with no cross-source metadata conflicts to resolve
- The goal is peak-list filtering or spectral similarity scoring rather than metadata
  field/identifier harmonization
- Spectra lack any SMILES/InChI or precursor m/z, so they will be discarded rather
  than normalized
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: FragHub
  role: Orchestrates the parsing, cleaning, and segregation workflow; provides the
    GUI through which input files, filters, and output formats are selected
  repo: https://github.com/eMetaboHUB/FragHub
- name: RDKit
  role: Standardizes SMILES/InChI/InChIKey and calculates exact and average molecular
    masses for each compound
- name: spectra-hash (SPLASH)
  role: Generates the unique SPLASH identifier recorded per spectrum and used to deduplicate
    records across updates
  repo: https://github.com/berlinguyinca/spectra-hash
- name: PubChem
  role: Source of fingerprint and ontology/descriptor data for compounds identified
    by InChIKey
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:9044a91dd2335a2dae5deee53237b307db8ec5d96493591ae721eefbf91d2f9d
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
  - FragHub
  - PubChem
  - Python
  - RDKit
  - spectra-hash (SPLASH)
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 20
---

# spectral-metadata-normalization-rdkit

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalizes and harmonizes mass spectral metadata from Open Mass Spectral Libraries (.MSP, .MGF, .JSON, .CSV, .XML) by standardizing chemical identifiers (SMILES, InChI, InChIKey) with RDKit, computing exact/average masses, and harmonizing instrument/ionization terms via HUPO PSI-MS controlled vocabulary before library integration.

## When to use

Use when integrating or standardizing spectra from heterogeneous OMSLs (GNPS .MGF, MoNA .MSP/.SDF/.JSON, MassBank, MS-Dial) so that spectra carry consistent identifiers, masses, and ontology terms for downstream annotation tools.

## When NOT to use

- Input is a single self-consistent library already using standardized fields and identifiers, with no cross-source metadata conflicts to resolve
- The goal is peak-list filtering or spectral similarity scoring rather than metadata field/identifier harmonization
- Spectra lack any SMILES/InChI or precursor m/z, so they will be discarded rather than normalized

## Inputs

- Spectral library files in .MSP, .MGF, .JSON, .CSV, or .XML format (e.g., GNPS-tagged .MGF, MoNA .MSP/.SDF/.JSON, MassBank records)
- Internal dictionaries mapping original metadata values to HUPO PSI-MS controlled-vocabulary terms
- PubChem-derived ontology/annotation data keyed by InChIKey

## Outputs

- Standardized spectra with normalized chemical identifiers (SMILES, InChI, InChIKey), recalculated exact/average masses, HUPO PSI-MS-harmonized metadata, and SPLASH keys
- Spectra segregated into directories/files by ionization mode, separation technique (LC/GC), and experimental vs predicted origin
- update.json file containing SPLASH keys of all processed spectra for incremental reprocessing

## How to apply

Parse each spectrum record, then use RDKit to verify/standardize SMILES, InChI, and InChIKey and to recalculate exact and average molecular masses from the structure. Harmonize instrument and ionization metadata into HUPO PSI-MS controlled-vocabulary terms using FragHub's internal dictionaries, and generate a SPLASH key for every spectrum. Segregate the cleaned spectra by polarity (positive/negative), chromatographic mode (LC/GC), and acquisition type (experimental/in silico); drop any spectrum lacking both InChI and SMILES, a valid precursor m/z, or adduct information. Keep a repository of processed SPLASH keys in an 'update.json' file so subsequent updates only process newly added spectra.

## Related tools

- **FragHub** (Orchestrates the parsing, cleaning, and segregation workflow; provides the GUI through which input files, filters, and output formats are selected) — https://github.com/eMetaboHUB/FragHub
- **RDKit** (Standardizes SMILES/InChI/InChIKey and calculates exact and average molecular masses for each compound)
- **spectra-hash (SPLASH)** (Generates the unique SPLASH identifier recorded per spectrum and used to deduplicate records across updates) — https://github.com/berlinguyinca/spectra-hash
- **PubChem** (Source of fingerprint and ontology/descriptor data for compounds identified by InChIKey)

## Examples

```
FragHub  # GUI: tab1 select .msp/.mgf/.json/.csv inputs; tab3 enable filters; tab5 choose .MSP/.JSON outputs; tab6 reuse update.json SPLASH-key repository
```

## Evaluation signals

- No spectra lacking both InChI and SMILES, or lacking precursor m/z/adduct information, appear in the output
- Every output spectrum carries a non-null SPLASH key and HUPO PSI-MS-compliant instrument/ionization field values
- Exact and average molecular masses in the output match values recalculated from the standardized structure by RDKit
- update.json contains SPLASH keys for all previously processed spectra; rerunning with the same input adds no duplicates
- Output records conform to GNPS naming conventions and are cleanly partitioned by polarity, LC/GC, and experimental/in silico flags

## Limitations

- Collision energies were not harmonized because of roughly 70 nonstandardized formats across libraries
- Inconsistent spectra lacking SMILES/InChI, precursor m/z, or adduct information are removed, so normalization is lossy for poorly annotated records
- The integrated coverage of 32,193 unique chemicals is low compared with the estimated millions of natural products, so many compounds fall outside the harmonized library
- Redundancy across OMSLs can create inconsistencies in downstream dot-product/reverse-dot-product scoring, which is sensitive to fragment number and intensity

## Evidence
