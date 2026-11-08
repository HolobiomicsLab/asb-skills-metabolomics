---
name: stereochemistry-preserving-molecule-merging
description: Use when merging molecular structures from multiple NP data sources into
  a non-redundant catalog where entries must be unified by stereochemistry-insensitive
  identity but original stereoisomers must remain retrievable under the parent entry.
when_to_use_negative:
- When stereoisomers must remain top-level distinct entries (this scheme deliberately
  collapses them under one stereochemistry-free parent)
- When molecules are synthetic reference compounds (the 'sm' set used only for NP-likeness
  score recalculation), not curated natural products
- When the input structures failed curation — RDKit-unparseable molecules and ChEMBL
  error code >= 6 molecules are rejected and must not enter the merge
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: RDKit
  role: Parses incoming structures and computes the standard InChI whose stereochemistry
    layers are stripped to derive the grouping key; unparseable molecules are eliminated
  repo: https://www.rdkit.org/
- name: CDK
  role: Processes chemical information and formats during structure curation and descriptor
    computation feeding the merge
- name: MongoDB
  role: Stores the staged sourceNaturalProduct records and the unified uniqueNaturalProduct
    collection, with hashed InChI/inchikey indexes supporting the deduplication lookup
- name: COCONUT compiler JAR
  role: Executes the from-scratch compilation including duplicate merging and CNP
    identifier assignment
  repo: https://github.com/mSorok/COCONUT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:d7e160bcf74059e017244cc940217c064fcbe8233221085a91c33a083db130a5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CDK
  - COCONUT compiler JAR
  - MongoDB
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# stereochemistry-preserving-molecule-merging

## Summary

Deduplicates natural product structures from multiple curated sources by grouping on stereochemistry-free InChI keys, merging duplicates under a single highly annotated parent entry while retaining the original stereochemical structures as nested variants, and assigning each unified entry a CNP-prefixed COCONUT identifier.

## When to use

Use when merging molecular structures from multiple NP data sources into a non-redundant catalog where entries must be unified by stereochemistry-insensitive identity but original stereoisomers must remain retrievable under the parent entry.

## When NOT to use

- When stereoisomers must remain top-level distinct entries (this scheme deliberately collapses them under one stereochemistry-free parent)
- When molecules are synthetic reference compounds (the 'sm' set used only for NP-likeness score recalculation), not curated natural products
- When the input structures failed curation — RDKit-unparseable molecules and ChEMBL error code >= 6 molecules are rejected and must not enter the merge

## Inputs

- Curated per-source natural product records with original molecular structures (SMILES/SDF/MOL from the COCONUT release 2024 PostgreSQL snapshot)
- Precomputed stereochemistry-free InChI keys per molecule
- Existing CNP identifier mapping file (e.g., coconut_ids_june2020.csv)

## Outputs

- Unified UniqueNaturalProduct MongoDB collection with one parent entry per stereochemistry-free InChI key and nested stereochemical variants
- CNP-prefixed 7-digit COCONUT identifiers (e.g., CNP0228556) with .0-style variant suffixes
- Structured JSON export of the unified collection

## How to apply

Load the curated per-source molecule records (e.g., from the COCONUT release 2024 PostgreSQL snapshot) into a MongoDB staging collection such as sourceNaturalProduct. For each pipeline-passed molecule, compute the standard InChI with RDKit/CDK, strip the stereochemistry layers, and derive the stereochemistry-free InChI key; index this key (e.g., db.uniqueNaturalProduct.createIndex({inchikey:1})). Group molecules by identical stereochemistry-free InChI key and merge duplicates into a single UniqueNaturalProduct entry, promoting the most highly annotated entry to parent and nesting the remaining entries' original stereochemical structures within it. Assign each unified NP a COCONUT identifier of the form 'CNP' plus 7 digits (e.g., CNP0228556), using the .0-style variant suffix to distinguish preserved stereochemical variants of the same parent; note the README warns that producing your own CNPid via 'addCNPid' is 'not recommended' — prefer importing existing IDs. Export the unified collection to structured JSON and validate uniqueness invariants before release.

## Related tools

- **RDKit** (Parses incoming structures and computes the standard InChI whose stereochemistry layers are stripped to derive the grouping key; unparseable molecules are eliminated) — https://www.rdkit.org/
- **CDK** (Processes chemical information and formats during structure curation and descriptor computation feeding the merge)
- **MongoDB** (Stores the staged sourceNaturalProduct records and the unified uniqueNaturalProduct collection, with hashed InChI/inchikey indexes supporting the deduplication lookup)
- **COCONUT compiler JAR** (Executes the from-scratch compilation including duplicate merging and CNP identifier assignment) — https://github.com/mSorok/COCONUT

## Examples

```
java -Xmx16288m -jar coconut-0.0.1-SNAPSHOT.jar data sm/sm.tsv fragments/fragment_without_sugar.txt fragments/fragment_with_sugar.txt importCOCONUTids coconut_ids_june2020.csv
```

## Evaluation signals

- No two entries in the unified collection share the same stereochemistry-free InChI key
- Every entry carries a unique 'CNP' + 7-digit identifier, with stereochemical variants retained under the parent using .0-style suffixes
- Catalog totals are consistent with the reported 695,133 chemical molecules (89,593 non-stereo and 555,897 stereo) and 415,768 unique parent structures
- The parent entry of each merged group is the most highly annotated one (per the 5-star annotation level)
- inchikey and coconut_id indexes exist on uniqueNaturalProduct and lookups resolve uniquely

## Limitations

- Identity is defined without stereochemistry, so stereoisomers are folded into one parent entry and only preserved as nested variants — queries keyed on absolute stereo will not match the parent directly
- Generating new CNPid with the 'addCNPid' option is explicitly 'not recommended' in the README; existing IDs should be imported instead
- Molecules failing RDKit parsing or carrying ChEMBL error code >= 6 are rejected/eliminated and never reach the merge step

## Evidence

- [intro] inchikey grouping: "Then, NPs from different provenance are unified based on the identity of their InChI keys without stereochemistry."
- [methods] duplicate merging rule: "Duplicates have been merged into one entry and the highly annotated entry has been made the parent entry, and the remainder is now included in the parent entry."
- [methods] catalog totals: "The database catalogs a total of 695,133 chemical molecules, with 89,593 being non-stereo (lacking stereoisomers) and 555,897 stereo molecules"
- [readme] identifier indexing in MongoDB: "db.uniqueNaturalProduct.createIndex( {inchikey:1})
db.uniqueNaturalProduct.createIndex( {clean_smiles: "hashed"})
db.uniqueNaturalProduct.createIndex( {molecular_formula:1})"
