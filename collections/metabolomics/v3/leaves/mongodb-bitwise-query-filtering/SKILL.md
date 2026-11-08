---
name: mongodb-bitwise-query-filtering
description: Use when a molecule collection is stored in MongoDB and you need to answer
  substructure queries by filtering on precomputed fingerprint bits, then confirming
  candidates with graph pattern matching.
when_to_use_negative:
- The task is similarity search, not substructure search — COCONUT handles that separately
  with an LSH-based approach in MongoDB following the ChEBML blog post
- The backing store is not MongoDB, so no native $allBitsSet bitwise query operator
  is available and fingerprints cannot be screened with it
- The collection has no precomputed fingerprints stored as BinData; computing them
  per query defeats the purpose of the indexed screen
tools:
- name: MongoDB
  role: Stores the collection and its PubChem fingerprints as BinData; the native
    $allBitsSet function performs the bitwise candidate filter
- name: CDK
  role: Confirms candidate matches via pattern-matching algorithms — Ullmann (default),
    Vento-Foggia, or depth-first — against the query subgraph
- name: Kotlin
  role: Back-end language implementing the search request handling between front-end,
    Spring layer and the database
  repo: https://github.com/mSorok/NaturalProductsOnline
- name: Java 11
  role: Back-end runtime alongside Kotlin in which the search logic and CDK calls
    are executed
  repo: https://github.com/mSorok/NaturalProductsOnline
- name: Spring
  role: Framework through which the back-end processes front-end search requests and
    communicates with MongoDB (Spring data)
  repo: https://github.com/mSorok/NaturalProductsOnline
- name: OpenChemLib
  role: Handles the chemical editor used to draw/enter the query structure for the
    search functions
  repo: https://github.com/mSorok/NaturalProductsOnline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:664ea3df0b44e21a96a4a9462a052c1d048c39cc97528ad89c4786a73007c262
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mSorok/NaturalProductsOnline
  tools:
  - CDK
  - Java 11
  - Kotlin
  - MongoDB
  - OpenChemLib
  - Spring
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mongodb-bitwise-query-filtering

## Summary

Implements substructure search on MongoDB by precomputing PubChem fingerprints as BinData byte lists and using the native $allBitsSet query to filter candidates, followed by CDK pattern-matching confirmation (Ullmann default). This makes chemical substructure search feasible on a document database without a specialized cheminformatics engine.

## When to use

Use when a molecule collection is stored in MongoDB and you need to answer substructure queries by filtering on precomputed fingerprint bits, then confirming candidates with graph pattern matching.

## When NOT to use

- The task is similarity search, not substructure search — COCONUT handles that separately with an LSH-based approach in MongoDB following the ChEBML blog post
- The backing store is not MongoDB, so no native $allBitsSet bitwise query operator is available and fingerprints cannot be screened with it
- The collection has no precomputed fingerprints stored as BinData; computing them per query defeats the purpose of the indexed screen

## Inputs

- Molecule collection in MongoDB with precomputed PubChem fingerprints stored as BinData byte lists
- Query molecule structure (e.g. SMILES/Molfile edited via the OpenChemLib chemical editor) to be fingerprinted

## Outputs

- List of molecules whose fingerprints contain all query fingerprint bits (candidates)
- Confirmed substructure matches with molecule identifiers written to substructure_search_results.json (application/json)

## How to apply

First compute the PubChem fingerprint for every molecule in the collection and store it as a list of bytes (BinData type in MongoDB); COCONUT indexes this field for fast access. For a query structure, compute the same PubChem fingerprint and issue a MongoDB query using the native $allBitsSet function so only documents whose BinData fingerprint contains all query bits are retained — because substructure matching implies bit-set containment, this screen produces no false negatives. Retrieve the surviving candidates and confirm true substructure matches with CDK pattern-matching, using the Ullmann algorithm as default (Vento-Foggia or depth-first search are available alternatives). Write confirmed matches with their molecule identifiers to the results file, served as application/json. The two-stage design (bitwise filter then exact graph match) is what makes chemical search tractable on top of MongoDB.

## Related tools

- **MongoDB** (Stores the collection and its PubChem fingerprints as BinData; the native $allBitsSet function performs the bitwise candidate filter)
- **CDK** (Confirms candidate matches via pattern-matching algorithms — Ullmann (default), Vento-Foggia, or depth-first — against the query subgraph)
- **Kotlin** (Back-end language implementing the search request handling between front-end, Spring layer and the database) — https://github.com/mSorok/NaturalProductsOnline
- **Java 11** (Back-end runtime alongside Kotlin in which the search logic and CDK calls are executed) — https://github.com/mSorok/NaturalProductsOnline
- **Spring** (Framework through which the back-end processes front-end search requests and communicates with MongoDB (Spring data)) — https://github.com/mSorok/NaturalProductsOnline
- **OpenChemLib** (Handles the chemical editor used to draw/enter the query structure for the search functions) — https://github.com/mSorok/NaturalProductsOnline

## Examples

```
db.uniqueNaturalProduct.createIndex( { pubchemBits : "hashed" } )
```

## Evaluation signals

- Every reported match passes a CDK pattern-matching check (Ullmann default) against the query subgraph — no unconfirmed candidates should appear in the output
- No true substructure match is missing from the candidate set: $allBitsSet containment must be a superset of the CDK-confirmed result set (the bit screen must not introduce false negatives)
- Fingerprints in the database are stored as BinData byte lists, and the pubchemBits field is indexed (verified via the collection's index list)
- Results are returned in application/json format with molecule identifiers included

## Limitations

- The PubChem fingerprint is a screen, not an exact matcher: all bit-set candidates must still be verified by CDK pattern matching, so the $allBitsSet stage alone returns false positives
- Similarity search requires a separate, memory-intensive LSH-based implementation in MongoDB adapted from the ChEBML blog post — the bitwise filter does not serve similarity queries
- The companion compiler step requires substantial memory (e.g. java -Xmx12288m for similarity calculation) and index restoration is recommended without mongorestore's --noIndexRestore omission, i.e. indexes must be recreated manually

## Evidence

- [intro] Fingerprints precomputed and stored as BinData in MongoDB: "their fingerprints of choice (in COCONUT are used the PubChem fingerprints) to be precomputed and stored as a list of bytes (BinData type in MongoDB)."
- [intro] $allBitsSet filtering followed by CDK pattern matching: [evidence span withheld — anchor status: not_found]
- [intro] Confirmation step runs CDK pattern matching after candidate retrieval: [evidence span withheld — anchor status: not_found]
- [readme] pubchemBits field is indexed in the uniqueNaturalProduct collection: [evidence span withheld — anchor status: external_doc]
