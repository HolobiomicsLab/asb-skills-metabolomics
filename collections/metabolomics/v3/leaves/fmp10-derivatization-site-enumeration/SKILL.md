---
name: fmp10-derivatization-site-enumeration
description: Use when annotating MALDI-MSI m/z features from a derivatizing-matrix
  experiment (e.
when_to_use_negative:
- The experiment uses a conventional matrix producing [M+H]+/[M-H]- ions rather than
  a derivatizing matrix — no functional-group filtering is needed
- You already have Met-ID's SQLite databases parsed from HMDB and LIPID MAPS and can
  query its built-in functional-group tables directly
- Input is raw MALDI-MSI imaging data rather than a structure database — this skill
  operates on candidate metabolite structures, not spectra
edam_operation: http://edamontology.org/operation_3281
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Met-ID
  role: 'Reference implementation: generates a subset of targets containing only functional
    groups specific to the derivatization agent from HMDB/LIPID MAPS SQLite databases'
  repo: https://github.com/pbjarterot/Met-ID
- name: RDKit
  role: Parses SMILES structures and performs SMARTS substructure matching to count
    derivatizable sites per compound
- name: HMDB
  role: Source database of metabolite structures filtered by derivatization-site count
- name: LIPID MAPS
  role: Second source database Met-ID parses for candidate structures
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:a61e7889ef2266bc9d20f3b576f5fd8b9488ccdbaa4c6493757d42c0bdc964c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pbjarterot/Met-ID
  tools:
  - HMDB
  - LIPID MAPS
  - Met-ID
  - RDKit
  - SQLite
  edam_operation: http://edamontology.org/operation_3281
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# fmp10-derivatization-site-enumeration

## Summary

Enumerates the number of FMP-10-derivatizable functional groups (phenolic hydroxyls, primary/secondary amines) per metabolite structure using RDKit SMARTS substructure matching, replicating how Met-ID filters HMDB and LIPID MAPS candidates before MS1 annotation of derivatized MALDI-MSI features.

## When to use

Use when annotating MALDI-MSI m/z features from a derivatizing-matrix experiment (e.g. FMP-10) and you need to restrict a structure database (HMDB, LIPID MAPS) to compounds bearing the functional groups targeted by the derivatization agent, or to validate expected derivatization multiplicities per compound.

## When NOT to use

- The experiment uses a conventional matrix producing [M+H]+/[M-H]- ions rather than a derivatizing matrix — no functional-group filtering is needed
- You already have Met-ID's SQLite databases parsed from HMDB and LIPID MAPS and can query its built-in functional-group tables directly
- Input is raw MALDI-MSI imaging data rather than a structure database — this skill operates on candidate metabolite structures, not spectra

## Inputs

- HMDB metabolite structure records (SMILES/SDF) as used by Met-ID
- LIPID MAPS structure records (SMILES/SDF)
- SMARTS pattern(s) defining functional groups targeted by the derivatizing matrix (FMP-10: phenolic hydroxyls, primary/secondary amines)

## Outputs

- CSV table (e.g. derivatization_site_counts.csv) with columns: accession, compound name, derivatization-site count
- Filtered derivatized-target subset of database compounds containing FMP-10-reactive functional groups

## How to apply

Download the public HMDB (and LIPID MAPS) structure records (SMILES/SDF) and parse each with RDKit into sanitized molecular objects, neutralizing salts and skipping unparseable entries. Apply SMARTS substructure matching to enumerate derivatizable sites per compound — phenolic hydroxyls and primary/secondary amines for FMP-10 — mirroring how Met-ID 'utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent'. Mark compounds with zero matches (e.g. the cyanoacrylate enbucrilate) as non-derivatizable and exclude them from the derivatized-target subset, retaining the site count for validation. Assemble a table of accession, compound name, and derivatization-site count, then validate against the paper's four-isomer example: enbucrilate = 0, p-octopamine = 2, norfenefrine = 2, dopamine = 3 (multi-site counts matter because Met-ID 'is able to identify multiple derivatization sites on the same molecule').

## Related tools

- **Met-ID** (Reference implementation: generates a subset of targets containing only functional groups specific to the derivatization agent from HMDB/LIPID MAPS SQLite databases) — https://github.com/pbjarterot/Met-ID
- **RDKit** (Parses SMILES structures and performs SMARTS substructure matching to count derivatizable sites per compound)
- **HMDB** (Source database of metabolite structures filtered by derivatization-site count)
- **LIPID MAPS** (Second source database Met-ID parses for candidate structures)

## Examples

```
from rdkit import Chem; mol = Chem.MolFromSmiles(smiles); n_sites = len(mol.GetSubstructMatches(Chem.MolFromSmarts('[OX2H]c'))) + len(mol.GetSubstructMatches(Chem.MolFromSmarts('[NX3;H2,H3][#6]')))
```

## Evaluation signals

- Four-isomer validation: enbucrilate = 0 sites (excluded), p-octopamine = 2, norfenefrine = 2, dopamine = 3
- Every compound with zero SMARTS matches is excluded from the derivatized-target subset but still appears in the output CSV with count 0
- Each site count is a non-negative integer and corresponds to the sum of matched phenolic hydroxyls plus primary/secondary amines
- Compounds lacking parseable SMILES are skipped and logged rather than silently assigned 0 sites
- Output CSV schema contains exactly accession, compound name, and derivatization-site count columns

## Limitations

- The FMP-10 derivatizing matrix was developed in-house and its availability/distribution to other users is not stated
- SMARTS patterns only capture the functional groups encoded for the matrix; the approach assumes the derivatization reaction is fully represented by those substructures
- Met-ID's bundled database files may not update on reinstall and must be removed manually (e.g. the com.farmbio.metid folder in AppData/Library Application Support) before reparsing

## Evidence

- [abstract] It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent.: "It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent."
- [discussion] Met-ID is based on the openly available HMDB and LIPID MAPS: "Met-ID is based on the openly available HMDB and LIPID MAPS"
- [discussion] Met-ID is also dependent on rdkit, a C++ framework for computational chemistry: [evidence span withheld — anchor status: not_found]
- [methods] The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix.: "The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix."
- [discussion] contains files parsing the HMDB and LIPID MAPS into SQLite databases that Met-ID can read: [evidence span withheld — anchor status: not_found]
