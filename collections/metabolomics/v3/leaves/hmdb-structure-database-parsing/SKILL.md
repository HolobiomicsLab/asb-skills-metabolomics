---
name: hmdb-structure-database-parsing
description: Use when annotating MALDI-MSI m/z features against a derivatizing matrix
  (e.g. FMP-10) and you need HMDB/LIPID MAPS structures filtered to compounds bearing
  matrix-reactive functional groups.
when_to_use_negative:
- The MALDI-MSI dataset uses a conventional matrix where annotation is based on common
  [M+H]+ / [M-H]- ions rather than on-tissue derivatization
- The target database is Metlin or GNPS (LC-MS oriented) rather than the HMDB/LIPID
  MAPS structures Met-ID parses into SQLite
- You already have a derivatized-candidate subset or site-count table and only need
  MS2 spectral comparison or mass recalibration
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Met-ID
  role: Consumes the parsed HMDB/LIPID MAPS SQLite database and applies the structural
    functional-group filter to generate the derivatization-agent-specific target subset
  repo: https://github.com/pbjarterot/Met-ID
- name: RDKit
  role: Sanitizes SMILES structures and performs the SMARTS substructure matching
    that enumerates derivatizable sites per compound
  repo: https://www.rdkit.org/
- name: HMDB
  role: Openly available source of metabolite structure records parsed into Met-ID's
    SQLite database
- name: LIPID MAPS
  role: Second openly available structure database parsed alongside HMDB for Met-ID
    annotation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:37914c539508362eefdb82172ecde717a47b9e28d3c7accddc77b233e214a7e1
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# hmdb-structure-database-parsing

## Summary

Parses HMDB (and LIPID MAPS) metabolite structure records into sanitized RDKit molecular objects and SQLite tables so that derivatizing-matrix annotation (e.g. FMP-10) can filter candidates by derivatizable functional groups.

## When to use

Use when annotating MALDI-MSI m/z features against a derivatizing matrix (e.g. FMP-10) and you need HMDB/LIPID MAPS structures filtered to compounds bearing matrix-reactive functional groups.

## When NOT to use

- The MALDI-MSI dataset uses a conventional matrix where annotation is based on common [M+H]+ / [M-H]- ions rather than on-tissue derivatization
- The target database is Metlin or GNPS (LC-MS oriented) rather than the HMDB/LIPID MAPS structures Met-ID parses into SQLite
- You already have a derivatized-candidate subset or site-count table and only need MS2 spectral comparison or mass recalibration

## Inputs

- HMDB metabolite structure records (SMILES or SDF, e.g. the HMDB(All) export)
- LIPID MAPS structure records
- Derivatization-matrix functional-group definition (e.g. FMP-10 SMARTS for phenolic hydroxyls and primary/secondary amines)

## Outputs

- SQLite database of HMDB and LIPID MAPS metabolites readable by Met-ID
- Per-compound derivatization-site counts (e.g. derivatization_site_counts.csv with accession, name, site count)
- Filtered subset of target molecules containing functional groups specific to the derivatization agent

## How to apply

Download the public HMDB metabolite structure records (SMILES/SDF) that Met-ID targets, then parse each entry with RDKit into sanitized molecular objects, neutralizing salts and skipping unparseable entries. Following the article's workflow, load the structures into SQLite — Met-ID ships files 'parsing the HMDB and LIPID MAPS into SQLite databases that Met-ID can read' — with the default target set 'HMDB(All)'. Enumerate derivatization sites by SMARTS substructure matching against the matrix's reactive groups (for FMP-10, phenolic hydroxyls and primary/secondary amines), so that, as the paper states, the software 'generates a subset of targets that contain only functional groups specific to the derivatization agent.' Mark compounds with zero matches (e.g. the cyanoacrylate enbucrilate) as non-derivatizable and exclude them from the derivatized-target subset, retaining per-compound site counts for validation. The parsed, annotated subset then feeds Met-ID's identify() mass matching for derivatized MALDI-MSI features.

## Related tools

- **Met-ID** (Consumes the parsed HMDB/LIPID MAPS SQLite database and applies the structural functional-group filter to generate the derivatization-agent-specific target subset) — https://github.com/pbjarterot/Met-ID
- **RDKit** (Sanitizes SMILES structures and performs the SMARTS substructure matching that enumerates derivatizable sites per compound) — https://www.rdkit.org/
- **HMDB** (Openly available source of metabolite structure records parsed into Met-ID's SQLite database)
- **LIPID MAPS** (Second openly available structure database parsed alongside HMDB for Met-ID annotation)

## Examples

```
python -c "from rdkit import Chem; m=Chem.MolFromSmiles('NCCc1ccc(O)c(O)c1'); print(len(m.GetSubstructMatches(Chem.MolFromSmarts('[NX3;!$(N*=*);!$(N[#6]=[O,N,S])]'))))"
```

## Evaluation signals

- Validation against the paper's four-isomer example: enbucrilate = 0 sites, p-octopamine = 2, norfenefrine = 2, dopamine = 3
- Every HMDB/LIPID MAPS entry yields a row in the SQLite metabolite table, with unparseable or salt-containing entries explicitly skipped/neutralized rather than silently dropped
- Derivatization-site counts are non-negative integers and compounds with 0 matches are excluded from the derivatized-target subset but retained in the count table
- Met-ID list_tables / list_metabolites / list_matrices / list_functional_groups / list_adducts return populated tables after parsing

## Limitations

- RDKit installation is described as nontrivial (often handled via anaconda) and no pinned environment or dependency versions are specified
- Reinstalling Met-ID does not update its bundled database files; the 'com.farmbio.metid' app-data folder must be removed manually before reinstalling
- Coverage is limited to HMDB and LIPID MAPS; Metlin/GNPS-wide LC-MS databases are not part of this parsing workflow

## Evidence

- [abstract] It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent.: "It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent."
- [discussion] contains files parsing the HMDB and LIPID MAPS into SQLite databases that Met-ID can read: [evidence span withheld — anchor status: not_found]
- [methods] The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix.: "The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix."
- [discussion] Met-ID is also dependent on rdkit, a C++ framework for computational chemistry: [evidence span withheld — anchor status: not_found]
- [readme] Met-ID has a particular focus on derivatizing matrices leading to other ions than the common [M+H]+ in positive mode and [M-H]- in negative mode.: "Met-ID has a particular focus on derivatizing matrices leading to other ions than the common [M+H]+ in positive mode and [M-H]- in negative mode."
