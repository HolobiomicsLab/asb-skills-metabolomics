---
name: pubchem-fingerprint-extraction
description: Use when you need to convert normalized InChIKey identifiers from OMSL
  compound records (e.g., FragHub's standardized .mgf/.msp/.csv/.xml output for GNPS,
  MoNA, MassBank, MS-Dial) into PubChem fingerprint feature vectors for downstream
  dimensionality-reduction or chemical-space mapping.
when_to_use_negative:
- Input compounds lack a valid InChIKey or normalized SMILES/InChI — FragHub strips
  inconsistent spectra missing SMILES and InChI, so fingerprinting cannot proceed
  without prior standardization
- The goal is spectral similarity (dot product / reverse dot product) rather than
  structural chemical-space comparison — fingerprints describe structure, not fragmentation
  behavior
- Working with collision-energy harmonized spectra or ionization-mode metadata rather
  than structural descriptors — this skill operates on chemical identifiers only
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: FragHub
  role: Produces the standardized, normalized InChIKey identifiers and bundles offline
    PubChem data used to complete ontologic information on InChIKey
  repo: https://github.com/eMetaboHUB/FragHub
- name: PubChem
  role: Source of the molecular fingerprints used as the chemical descriptor for each
    InChIKey
- name: RDKit
  role: Used by FragHub to standardize chemical identifiers and calculate molecular
    masses, ensuring InChIKeys are valid before fingerprint retrieval
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:9502aabe9503bccc3a2f99aeb41cbb137e1d3136b440521dfe83209d8ef428d0
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
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pubchem-fingerprint-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts PubChem molecular fingerprints from normalized InChIKey identifiers harvested from FragHub-standardized OMSL compound records, providing the high-dimensional chemical descriptor matrix needed for t-SNE chemical-space visualization of mass spectral library coverage.

## When to use

Use when you need to convert normalized InChIKey identifiers from OMSL compound records (e.g., FragHub's standardized .mgf/.msp/.csv/.xml output for GNPS, MoNA, MassBank, MS-Dial) into PubChem fingerprint feature vectors for downstream dimensionality-reduction or chemical-space mapping.

## When NOT to use

- Input compounds lack a valid InChIKey or normalized SMILES/InChI — FragHub strips inconsistent spectra missing SMILES and InChI, so fingerprinting cannot proceed without prior standardization
- The goal is spectral similarity (dot product / reverse dot product) rather than structural chemical-space comparison — fingerprints describe structure, not fragmentation behavior
- Working with collision-energy harmonized spectra or ionization-mode metadata rather than structural descriptors — this skill operates on chemical identifiers only

## Inputs

- FragHub-standardized compound records (.mgf, .msp, .csv, .json, .xml output files) carrying normalized InChIKey per compound
- Offline PubChem descriptor data bundled with FragHub, or live PubChem PUG-REST fingerprint endpoint keyed by InChIKey
- Optional NPClassifier annotations (PathwayNP, superclassNP) per compound for figure coloration

## Outputs

- PubChem fingerprint feature matrix (one binary fingerprint vector per unique InChIKey)
- Per-compound mapping table linking InChIKey → fingerprint → originating OMSL label
- Input matrix suitable for t-SNE projection of OMSL chemical-space coverage

## How to apply

First ensure each compound carries a standardized InChIKey, produced by FragHub's RDKit-based recalculation and normalization of SMILES, InChI, and InChIKey fields; spectra lacking SMILES/InChI/InChIKey are removed by FragHub during processing, so only compounds surviving that cleanup should be fingerprinted. Retrieve the PubChem fingerprint for each InChIKey using the PubChem service (FragHub ships offline PubChem data and completes ontologic information on InChIKey), using the fingerprint block as the molecular descriptor vector. Optionally color-code compounds by NPClassifier PathwayNP or superclassNP for later visualization. Run the fingerprint extraction over the pooled compound set labeled by originating library (FragHub vs GNPS vs MoNA vs MassBank vs MS-Dial-VS17) so that each OMSL's occupied chemical space can be compared; the paper grounds this workflow by noting that dimensionality reduction 'was calculated from PubChem fingerprints'. Verify coverage against per-library compound counts: 13,507 GNPS, 21,839 MoNA, 8,358 MassBank, 22,282 MS-Dial-VS17, and 32,193 FragHub unique chemicals.

## Related tools

- **FragHub** (Produces the standardized, normalized InChIKey identifiers and bundles offline PubChem data used to complete ontologic information on InChIKey) — https://github.com/eMetaboHUB/FragHub
- **PubChem** (Source of the molecular fingerprints used as the chemical descriptor for each InChIKey)
- **RDKit** (Used by FragHub to standardize chemical identifiers and calculate molecular masses, ensuring InChIKeys are valid before fingerprint retrieval)

## Examples

```
from FragHub.scripts import standardize; std = standardize(lib_files=['GNPS.mgf','MoNA.msp','MassBank.xml','MSDial-VS17.csv']); fingerprints = [pubchem_fp(ik) for ik in std['inchikey']]
```

## Evaluation signals

- The number of compounds successfully fingerprinted should match the per-library counts reported in the paper: 13,507 GNPS, 21,839 MoNA, 8,358 MassBank, 22,282 MS-Dial-VS17, and 32,193 FragHub unique chemicals
- Every fingerprinted compound must possess a normalized InChIKey — no fingerprint entries should exist for compounds FragHub flagged as lacking SMILES and InChI
- The fingerprint matrix dimensions should equal the number of unique InChIKeys across the pooled libraries (35,673 raw identifiers reduced to 32,193 unique chemicals after FragHub refinement)
- Fingerprints derived from the offline PubChem data bundled in FragHub should match live PubChem fingerprint retrieval for spot-checked InChIKeys
- The extracted fingerprints, when fed to t-SNE (perplexity=30, exaggeration=1), should produce a 2-D projection consistent with the paper's Figure showing GNPS/MS-Dial denser in terpenoid/alkaloid regions and MoNA spanning broader carbohydrate space

## Limitations

- Only compounds with valid standardized InChIKey can be fingerprinted; FragHub explicitly removes spectra deemed inconsistent because they lack SMILES and InChI, precursor m/z, and adduct information
- No software version numbers are stated for PubChem data used in the workflow, so reproducibility of the offline PubChem fingerprint snapshot is uncertain
- Chemical class disparity between OMSLs and natural product catalogues means fingerprints alone under-represent natural-product chemical space relative to estimated millions of molecules
- Offline PubChem data bundled with FragHub may lag live PubChem updates, potentially altering fingerprint availability over time

## Evidence
