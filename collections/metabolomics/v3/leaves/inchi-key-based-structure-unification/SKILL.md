---
name: inchi-key-based-structure-unification
description: Use when curating or merging chemical structure collections from multiple
  provenance where duplicate/same-core molecules must be de-duplicated into one canonical
  entry while retaining stereochemical variants.
when_to_use_negative:
- When the goal is to preserve every stereochemical or tautomeric duplicate as a separate
  top-level record — the compiler's whole point is collapsing them under one parent
  entry
- When input molecules fail ChEMBL/RDKit parsing or carry ChEMBL error code ≥ 6 —
  those are rejected upstream and never reach the unification step
- When you only need exact-structure matching including stereochemistry (originalInchiKey),
  not the stereochemistry-free core grouping this skill performs
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RDKit
  role: Compute standard InChI and derive stereochemistry-free InChIKey per molecule;
    molecules failing to parse are eliminated before unification
  repo: https://www.rdkit.org/
- name: CDK
  role: Process chemical information and formats during the InChIKey computation pipeline
- name: MongoDB
  role: Store sourceNaturalProduct and uniqueNaturalProduct collections; hashed and
    regular indexes on simpleInchiKey, originalInchiKey, and coconut_id support the
    grouping and lookup
  repo: https://github.com/mSorok/NaturalProductsOnline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:f88fb0a2ae5dd8c559b5e926c18de1181f87a4453b422c5f309ffebd97cf2998
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CDK
  - MongoDB
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# inchi-key-based-structure-unification

## Summary

Unifies molecular structures originating from multiple open natural-product sources into single UniqueNaturalProduct entries by grouping on InChIKey identity stripped of stereochemistry, preserving original stereochemical structures as nested variants and assigning CNP identifiers.

## When to use

Use when curating or merging chemical structure collections from multiple provenance where duplicate/same-core molecules must be de-duplicated into one canonical entry while retaining stereochemical variants.

## When NOT to use

- When the goal is to preserve every stereochemical or tautomeric duplicate as a separate top-level record — the compiler's whole point is collapsing them under one parent entry
- When input molecules fail ChEMBL/RDKit parsing or carry ChEMBL error code ≥ 6 — those are rejected upstream and never reach the unification step
- When you only need exact-structure matching including stereochemistry (originalInchiKey), not the stereochemistry-free core grouping this skill performs

## Inputs

- Curated per-source molecular structures (SMILES/SDF/MOL) staged in a MongoDB sourceNaturalProduct collection from the COCONUT 2024 PostgreSQL snapshot
- Per-molecule stereochemistry-free InChIKey values (simpleInchiKey) computed with RDKit or CDK
- COCONUT id list file (coconut_ids_june2020.csv) for onlyImportCoconutIds mode

## Outputs

- MongoDB uniqueNaturalProduct collection of unified entries keyed by stereochemistry-free InChIKey with nested stereochemical variants
- COCONUT identifiers of the form CNP<7 digits> (e.g. CNP0228556) with .0-style variant suffixes for stereochemical variants
- Structured JSON export of the UniqueNaturalProduct collection

## How to apply

Load the curated per-source structures from the PostgreSQL snapshot of the COCONUT 2024 release into a MongoDB staging collection (sourceNaturalProduct). For every molecule compute the standard InChI with RDKit/CDK, strip the stereochemistry layers, and derive the stereochemistry-free InChIKey (stored as simpleInchiKey). Group molecules sharing an identical stereochemistry-free InChIKey and merge duplicates into one entry, promoting the most highly annotated entry (highest annotationLevel on the 5-star system) to the parent entry and nesting the remaining entries' original stereochemical structures under the parent, reachable via a .0-style variant suffix. Assign each unified entry a COCONUT identifier of the form 'CNP' followed by 7 digits (e.g. CNP0228556) via the compiler's addCNPid / onlyImportCoconutIds modes. Export the resulting uniqueNaturalProduct collection to JSON for downstream annotation and release.

## Related tools

- **RDKit** (Compute standard InChI and derive stereochemistry-free InChIKey per molecule; molecules failing to parse are eliminated before unification) — https://www.rdkit.org/
- **CDK** (Process chemical information and formats during the InChIKey computation pipeline)
- **MongoDB** (Store sourceNaturalProduct and uniqueNaturalProduct collections; hashed and regular indexes on simpleInchiKey, originalInchiKey, and coconut_id support the grouping and lookup) — https://github.com/mSorok/NaturalProductsOnline

## Examples

```
java -Xmx16288m -jar coconut-0.0.1-SNAPSHOT.jar data sm/sm.tsv fragments/fragment_without_sugar.txt fragments/fragment_with_sugar.txt importCOCONUTids coconut_ids_june2020.csv
```

## Evaluation signals

- No two entries in the uniqueNaturalProduct collection share the same stereochemistry-free InChIKey (uniqueness invariant)
- Every entry carries a unique CNP-prefixed 7-digit identifier, and stereochemical variants are retained nested under the parent via the .0-style suffix
- Catalog totals consistent with the reported 695,133 molecules (89,593 non-stereo + 555,897 stereo) and 415,768 unique parent structures
- Every unified entry resolves to at least one sourceNaturalProduct record grouped under it (no orphan parents)
- Indexes on simpleInchiKey and coconut_id exist and match the mongo shell createIndex commands in the README

## Limitations

- The compiler JAR must be run with substantial memory (e.g. -Xmx16288m) and the similarity-computation step is noted as a large-memory operation that may need to run separately
- COCONUT id assignment in isolation (addCNPid) is explicitly 'not recommended' for producing your own identifiers; the recommended path is importing from a prepared CSV
- Molecules that fail RDKit parsing or reach ChEMBL error code 6+ are silently excluded from unification and only kept for future reference, so counts will differ from raw source totals

## Evidence

- [intro] Unify on stereo-free InChIKey: "Then, NPs from different provenance are unified based on the identity of their InChI keys without stereochemistry."
- [methods] Duplicates merged into parent: "Duplicates have been merged into one entry and the highly annotated entry has been made the parent entry"
- [readme] CNP identifier format: [evidence span withheld — anchor status: not_found]
- [readme] Indexes on InChIKey fields: [evidence span withheld — anchor status: external_doc]
- [methods] Catalog totals: "The database catalogs a total of 695,133 chemical molecules, with 89,593 being non-stereo"
