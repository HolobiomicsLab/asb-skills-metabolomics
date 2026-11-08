---
name: four-isomer-validation-benchmarking
description: Use when implementing or testing a filter that subsets metabolite databases
  (HMDB, LIPID MAPS) by derivatization-agent-reactive functional groups and you need
  known ground-truth site counts to verify correctness.
when_to_use_negative:
- The downstream task is MS1 mass matching or MS2 spectral similarity comparison of
  already-derivatized m/z features, not enumerating reactive functional groups on
  database structures
- The matrix is a conventional (non-derivatizing) MALDI matrix, so no functional-group-specific
  target subsetting applies
- Candidate molecules have no parseable structural representation (SMILES/SDF), since
  SMARTS matching cannot be performed
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RDKit
  role: Parses HMDB/LIPID MAPS structures and performs SMARTS substructure matching
    to enumerate derivatizable sites per compound
  repo: https://www.rdkit.org/
- name: Met-ID
  role: Reference implementation whose derivatization-matrix target subsetting and
    multi-site identification are being benchmarked; built on RDKit
  repo: https://github.com/pbjarterot/Met-ID
- name: HMDB
  role: Openly available source database of metabolite structures supplying the candidate
    compounds for site counting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:7cb6b5330124183af90753d687490840ead4ac164e230a0ea587518b97c47162
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMDB
  - Met-ID
  - RDKit
  - SQLite
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# four-isomer-validation-benchmarking

## Summary

Validates a derivatization-site enumeration pipeline (e.g. Met-ID's FMP-10 functional-group filter) by reproducing the paper's four-isomer benchmark, where RDKit substructure counting must yield 0 sites for enbucrilate, 2 for p-octopamine, 2 for norfenefrine, and 3 for dopamine.

## When to use

Use when implementing or testing a filter that subsets metabolite databases (HMDB, LIPID MAPS) by derivatization-agent-reactive functional groups and you need known ground-truth site counts to verify correctness.

## When NOT to use

- The downstream task is MS1 mass matching or MS2 spectral similarity comparison of already-derivatized m/z features, not enumerating reactive functional groups on database structures
- The matrix is a conventional (non-derivatizing) MALDI matrix, so no functional-group-specific target subsetting applies
- Candidate molecules have no parseable structural representation (SMILES/SDF), since SMARTS matching cannot be performed

## Inputs

- HMDB metabolite structure records (SMILES/SDF) parsed by Met-ID
- LIPID MAPS structure records (SMILES/SDF)
- SMARTS patterns for derivatization-agent-reactive functional groups (phenolic OH, primary/secondary amines for FMP-10)
- Ground-truth four-isomer site counts: enbucrilate=0, p-octopamine=2, norfenefrine=2, dopamine=3

## Outputs

- derivatization_site_counts.csv with columns accession, compound name, derivatization-site count
- Binary derivatizable/non-derivatizable classification per compound (zero-match compounds excluded from the derivatized-target subset)
- Pass/fail validation report against the four-isomer benchmark

## How to apply

Download the HMDB structure records used by Met-ID and parse each SMILES/SDF entry with RDKit into sanitized molecules, neutralizing salts and skipping unparseable entries. Define SMARTS patterns capturing the FMP-10-reactive groups — phenolic hydroxyls and primary/secondary amines — mirroring how Met-ID 'generates a subset of targets that contain only functional groups specific to the derivatization agent'. Enumerate substructure matches per compound with GetSubstructMatches, recording zero-match compounds (e.g. the cyanoacrylate enbucrilate) as non-derivatizable and excluded from the derivatized-target subset. Then validate against the paper's four-isomer example as a regression test: the filter must exclude enbucrilate (0 sites) while assigning p-octopamine = 2, norfenefrine = 2, and dopamine = 3 sites, since 'The software is able to identify multiple derivatization sites on the same molecule ... exemplified by FMP-10'. Emit a table of accession, compound name, and site count (derivatization_site_counts.csv) so counts remain auditable for any compound the filter retains.

## Related tools

- **RDKit** (Parses HMDB/LIPID MAPS structures and performs SMARTS substructure matching to enumerate derivatizable sites per compound) — https://www.rdkit.org/
- **Met-ID** (Reference implementation whose derivatization-matrix target subsetting and multi-site identification are being benchmarked; built on RDKit) — https://github.com/pbjarterot/Met-ID
- **HMDB** (Openly available source database of metabolite structures supplying the candidate compounds for site counting)

## Examples

```
from rdkit import Chem; m=Chem.MolFromSmiles('NCCc1ccc(O)c(O)c1'); print(len(m.GetSubstructMatches(Chem.MolFromSmarts('[OX2H]c'))) + len(m.GetSubstructMatches(Chem.MolFromSmarts('[NX3;H2,H3][#6]'))))  # dopamine -> 3
```

## Evaluation signals

- Enbucrilate (cyanoacrylate) returns exactly 0 derivatizable sites and is excluded from the derivatized-target subset
- p-octopamine and norfenefrine each return exactly 2 sites; dopamine returns exactly 3
- Every output row has a non-null accession, compound name, and integer site count >= 0
- All zero-match and multi-match compounds are reproducible when the SMARTS patterns are re-run on the same HMDB release
- Site counts are symmetric with known chemistry: every counted site corresponds to a phenolic hydroxyl or primary/secondary amine match

## Limitations

- FMP-10 was developed in-house, so benchmark datasets and the matrix itself may not be readily available to other users
- The SMARTS definition of 'derivatizable' sites is an approximation; ambiguity in phenolic vs. aliphatic hydroxyl or amine protonation state can shift counts
- RDKit itself is noted as a nontrivial installation (often via anaconda) with no pinned environment/dependency versions specified, so results may vary across toolkit versions
- This skill validates the filtering step only; it does not cover the paper's noted limitation of a finite preconfigured MS2 spectral database

## Evidence

- [abstract] It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent.: "It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent."
- [abstract] The software is able to identify multiple derivatization sites on the same molecule, facilitating identification of the derivatized compound. This ability is exemplified by FMP-10: "The software is able to identify multiple derivatization sites on the same molecule, facilitating identification of the derivatized compound. This ability is exemplified by FMP-10"
- [readme] Met-ID has a particular focus on derivatizing matrices leading to other ions than the common [M+H]+ in positive mode and [M-H]- in negative mode.: "Met-ID has a particular focus on derivatizing matrices leading to other ions than the common [M+H]+ in positive mode and [M-H]- in negative mode."
