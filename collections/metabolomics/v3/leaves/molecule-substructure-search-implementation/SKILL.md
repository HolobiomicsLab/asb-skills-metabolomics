---
name: molecule-substructure-search-implementation
description: Use when building a chemical search backend on a non-relational store
  (e.g. MongoDB) and you need exact substructure matching over large molecule collections
  without a dedicated cheminformatics search engine.
when_to_use_negative:
- When the goal is similarity rather than exact substructure search — COCONUT handles
  that separately with an LSH-based similarity search in MongoDB and should not reuse
  the $allBitsSet screen.
- When a dedicated cheminformatics search engine (e.g. a cartridge with native substructure
  indexing) is available; the MongoDB $allBitsSet screen is a workaround for chemical
  search being uncommon with MongoDB.
- When molecules lack parseable structures or valid fingerprints (e.g. structures
  the ChEMBL curation pipeline cannot parse) — candidates cannot be screened reliably.
edam_operation: http://edamontology.org/operation_3561
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3314
tools:
- name: MongoDB
  role: Stores molecules and their PubChem fingerprints as BinData; the native $allBitsSet
    operator performs the bitwise candidate screen
- name: CDK
  role: Pattern-matching library that confirms fingerprint-screened candidates as
    true substructure matches (Ullmann default; Vento-Foggia or depth-first alternatives)
- name: OpenChemLib
  role: Handles the chemical editor used to draw/submit the query structure for the
    search functions
- name: Kotlin
  role: Back-end language implementing the search request processing and database
    communication
- name: Spring
  role: Framework hosting the back-end that exposes the substructure search as an
    application/json service
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:64e1777ac0186103a16243393ee81afd836c4da109d9dd7bad6702170f12b61c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CDK
  - Java 11
  - Kotlin
  - MongoDB
  - OpenChemLib
  - Spring
  edam_operation: http://edamontology.org/operation_3561
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# molecule-substructure-search-implementation

## Summary

Implements chemical substructure search on top of MongoDB by precomputing PubChem fingerprints for each molecule, storing them as BinData, filtering candidates with the native $allBitsSet operator, and confirming hits with CDK graph-isomorphism pattern matching. This two-stage screen-then-verify design makes fingerprint-based substructure search feasible in a document database.

## When to use

Use when building a chemical search backend on a non-relational store (e.g. MongoDB) and you need exact substructure matching over large molecule collections without a dedicated cheminformatics search engine.

## When NOT to use

- When the goal is similarity rather than exact substructure search — COCONUT handles that separately with an LSH-based similarity search in MongoDB and should not reuse the $allBitsSet screen.
- When a dedicated cheminformatics search engine (e.g. a cartridge with native substructure indexing) is available; the MongoDB $allBitsSet screen is a workaround for chemical search being uncommon with MongoDB.
- When molecules lack parseable structures or valid fingerprints (e.g. structures the ChEMBL curation pipeline cannot parse) — candidates cannot be screened reliably.

## Inputs

- Molecule collection in MongoDB (e.g. uniqueNaturalProduct documents with structure fields such as clean_smiles, inchi, inchikey)
- Query molecule or query subgraph structure (provided via the OpenChemLib chemical editor on the front-end)
- Precomputed PubChem fingerprints stored as BinData per molecule document

## Outputs

- substructure_search_results.json containing confirmed substructure matches with molecule identifiers
- HTTP response in application/json format from the Spring back-end listing matching COCONUT entries

## How to apply

First, compute PubChem fingerprints for every molecule in the collection and store each as a list of bytes (BinData type) in the MongoDB document, indexing the fingerprint field (e.g. pubchemBits) for fast lookup. At query time, compute the same PubChem fingerprint for the query subgraph and issue a MongoDB filter using the native $allBitsSet operator so only molecules whose BinData fingerprint has all query bits set are returned; this conservative screen guarantees no true match is missed since a substructure's fingerprint bits are a subset of the superstructure's. Then confirm each candidate with the CDK pattern-matching library, using the Ullmann algorithm as default and Vento-Foggia or depth-first search as alternatives, comparing the query subgraph against candidate structures. Finally, write confirmed matches with molecule identifiers to substructure_search_results.json and serve results as application/json from the Kotlin/Java 11 Spring back-end. The rationale for the two-stage design is that $allBitsSet is a cheap bitwise screen that shrinks the candidate set before the expensive per-molecule CDK isomorphism check.

## Related tools

- **MongoDB** (Stores molecules and their PubChem fingerprints as BinData; the native $allBitsSet operator performs the bitwise candidate screen)
- **CDK** (Pattern-matching library that confirms fingerprint-screened candidates as true substructure matches (Ullmann default; Vento-Foggia or depth-first alternatives))
- **OpenChemLib** (Handles the chemical editor used to draw/submit the query structure for the search functions)
- **Kotlin** (Back-end language implementing the search request processing and database communication)
- **Spring** (Framework hosting the back-end that exposes the substructure search as an application/json service)

## Examples

```
mongorestore --port 27022 --db=COCONUT_2021_03 --noIndexRestore . && mongo --port 27022 --eval 'db.uniqueNaturalProduct.createIndex( { pubchemBits : "hashed" } )'
```

## Evaluation signals

- Every returned match passes the CDK pattern-matching check against the query subgraph (no false positives survive the confirmation stage).
- Results are served in application/json format.
- Confirmed matches are written to substructure_search_results.json with molecule identifiers.
- No true substructure is missed: any molecule whose PubChem fingerprint contains all query bits must appear in the candidate set (the $allBitsSet screen is a superset filter).
- Database restore/index step completes with the pubchemBits hashed index created on uniqueNaturalProduct (per README: db.uniqueNaturalProduct.createIndex( { pubchemBits : "hashed" } )).

## Limitations

- The $allBitsSet screen is necessary but not sufficient — every candidate still requires an expensive CDK isomorphism check, so throughput depends on fingerprint selectivity.
- Substructure search is uncommon with MongoDB; the approach relies on precomputed fingerprints and would need re-indexing if the fingerprint scheme changes.
- Similarity search is handled by a different mechanism (LSH-based, following a ChEBML blog post adapted to Java, Kotlin and Spring data), so the two search modes must be maintained separately.
- The full pipeline requires substantial memory for related computations (the README notes the similarity step 'requires a large amount of memory (this will be optimized in the future)'), and databases should be restored without indexes then rebuilt (mongorestore --noIndexRestore).

## Evidence

- [intro] Fingerprint precomputation and storage: "their fingerprints of choice (in COCONUT are used the PubChem fingerprints) to be precomputed and stored as a list of bytes (BinData type in MongoDB)."
- [intro] Confirmation step with CDK: "The CDK [13] library is used to process chemical information and formats."
- [intro] Back-end stack: "The COCONUT back-end, allowing to process the front-end requests and to communicate with the database is written in Kotlin and Java 11"
- [readme] pubchemBits index in README: [evidence span withheld — anchor status: external_doc]
- [intro] Similarity search uses a different mechanism: "Similarity search with MongoDB was implemented following the excellent ChEBML blog post on LSH-based similarity search in MongoDB [32] and adapted to Java, Kotlin and Spring data."
