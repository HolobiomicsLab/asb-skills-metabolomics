---
name: molecular-fingerprint-precomputation
description: Use when building substructure or similarity search over a large chemical
  database in a store without native chemistry support (e.g. MongoDB), where per-query
  on-the-fly fingerprinting would be too slow.
when_to_use_negative:
- The database engine natively supports chemical substructure search (e.g. RDKit PostgreSQL
  cartridges), making manual fingerprint precomputation and $allBitsSet filtering
  unnecessary
- The task is exact structure lookup by InChIKey or hashed SMILES index rather than
  substructure screening — COCONUT uses direct indexes on inchikey, clean_smiles etc.
  for that
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: CDK
  role: Computes PubChem fingerprints for each molecule structure and provides the
    Ullmann, Vento-Foggia and depth-first pattern-matching algorithms used to confirm
    candidates after the $allBitsSet filter
- name: MongoDB
  role: Stores the precomputed fingerprints as BinData and evaluates the native $allBitsSet
    operator to filter candidate molecules
- name: Kotlin
  role: Back-end language (with Java 11 and Spring) implementing the search pipeline
    that derives query fingerprints and runs CDK confirmation
  repo: https://github.com/mSorok/NaturalProductsOnline
- name: Java 11
  role: Back-end runtime, together with Kotlin and Spring, for processing search requests
    and communicating with MongoDB
  repo: https://github.com/mSorok/NaturalProductsOnline
- name: Spring
  role: Framework used by the back-end to adapt the LSH-based/fingerprint-based MongoDB
    search workflow to Java, Kotlin and Spring data
  repo: https://github.com/mSorok/NaturalProductsOnline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:924c796ce1c220c0f1d8ac8f4b418cc72300c5bf4094afc3d6e2352859e76ad5
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# molecular-fingerprint-precomputation

## Summary

Precompute chemical fingerprints (e.g. PubChem fingerprints) for every molecule in a large collection and store them as binary data (MongoDB BinData) so that database-native bit operations can serve as a fast pre-filter for substructure and similarity search before expensive graph matching.

## When to use

Use when building substructure or similarity search over a large chemical database in a store without native chemistry support (e.g. MongoDB), where per-query on-the-fly fingerprinting would be too slow.

## When NOT to use

- The database engine natively supports chemical substructure search (e.g. RDKit PostgreSQL cartridges), making manual fingerprint precomputation and $allBitsSet filtering unnecessary
- The task is exact structure lookup by InChIKey or hashed SMILES index rather than substructure screening — COCONUT uses direct indexes on inchikey, clean_smiles etc. for that

## Inputs

- Molecule structure records (e.g. SMILES/MOL parsed via CDK) loaded into a MongoDB collection such as uniqueNaturalProduct
- Query molecular structure for which a subgraph fingerprint must be derived

## Outputs

- Per-molecule PubChem fingerprint stored as BinData (list of bytes) in MongoDB, with indexes on pubchemBits and pfCounts.bits/pfCounts.count
- Set of candidate molecules passing the $allBitsSet filter, subsequently confirmed as true substructure matches by CDK pattern matching

## How to apply

Compute a chosen fingerprint (COCONUT uses PubChem fingerprints via the CDK library) for each structure in the collection and store it as a list of bytes, using the BinData type in MongoDB; index the fingerprint field (e.g. a hashed index on pubchemBits as shown in the COCONUT README). At query time, derive the query molecule's subgraph fingerprint and use MongoDB's native $allBitsSet operator to retain only candidate molecules whose stored fingerprint has all query bits set. Because bit-set filtering is necessary but not sufficient for true substructure containment, confirm candidates with a chemical pattern-matching algorithm (COCONUT defaults to Ullmann, with Vento-Foggia or depth-first alternatives) from the CDK. The rationale is that a precomputed, indexed binary fingerprint turns substructure search into an inexpensive bit comparison inside the database, drastically shrinking the candidate set before the expensive isomorphism check runs in the application layer.

## Related tools

- **CDK** (Computes PubChem fingerprints for each molecule structure and provides the Ullmann, Vento-Foggia and depth-first pattern-matching algorithms used to confirm candidates after the $allBitsSet filter)
- **MongoDB** (Stores the precomputed fingerprints as BinData and evaluates the native $allBitsSet operator to filter candidate molecules)
- **Kotlin** (Back-end language (with Java 11 and Spring) implementing the search pipeline that derives query fingerprints and runs CDK confirmation) — https://github.com/mSorok/NaturalProductsOnline
- **Java 11** (Back-end runtime, together with Kotlin and Spring, for processing search requests and communicating with MongoDB) — https://github.com/mSorok/NaturalProductsOnline
- **Spring** (Framework used by the back-end to adapt the LSH-based/fingerprint-based MongoDB search workflow to Java, Kotlin and Spring data) — https://github.com/mSorok/NaturalProductsOnline

## Examples

```
mongo --port 27022 use COCONUT_2021_03; db.uniqueNaturalProduct.createIndex({ pubchemBits: "hashed" }); db.uniqueNaturalProduct.find({ pubchemFingerprint: { $allBitsSet: <queryBits> } })
```

## Evaluation signals

- Every confirmed hit must pass the CDK pattern-matching check (Ullmann by default) against the query subgraph — bit-filter survivors that fail isomorphism testing are false positives
- The stored fingerprint field is of MongoDB BinData type and the pubchemBits index exists (verify with db.uniqueNaturalProduct.getIndexes())
- Applying $allBitsSet with a query fingerprint yields a candidate set that is a superset of the final confirmed substructure matches (no confirmed match may be absent from the filter result)
- Results of the confirmed substructure search are returned in application/json format with molecule identifiers

## Limitations

- The $allBitsSet bit filter is a necessary-but-not-sufficient screen: molecules whose fingerprint contains all query bits may still fail true substructure matching, so an expensive CDK confirmation step remains mandatory
- Similarity search in MongoDB required a separate, memory-heavy computation in the COCONUT compiler (runOnlySimilarity with -Xmx12288m); the authors note this 'will be optimized in the future'
- Fingerprints must be recomputed or backfilled whenever new structures are ingested (e.g. via the recomputeMissing mode of the COCONUT compiler JAR)

## Evidence

- [intro] Fingerprint precomputation and BinData storage: "their fingerprints of choice (in COCONUT are used the PubChem fingerprints) to be precomputed and stored as a list of bytes (BinData type in MongoDB)."
- [intro] $allBitsSet filtering and CDK confirmation: [evidence span withheld — anchor status: not_found]
- [readme] Hashed index on fingerprint bits in README: [evidence span withheld — anchor status: external_doc]
- [intro] CDK role in processing chemical information: "The CDK [13] library is used to process chemical information and formats."
- [intro] Similarity search adaptation to the Java/Kotlin/Spring stack: "Similarity search with MongoDB was implemented following the excellent ChEBML blog post on LSH-based similarity search in MongoDB [32] and adapted to Java, Kotlin and Spring data."
