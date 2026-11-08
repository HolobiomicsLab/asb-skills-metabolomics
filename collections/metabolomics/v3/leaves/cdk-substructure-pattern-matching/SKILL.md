---
name: cdk-substructure-pattern-matching
description: Use when a query molecule's subgraph fingerprint has already narrowed
  candidates (e.g. via $allBitsSet on PubChem fingerprints stored as BinData in MongoDB)
  and you must confirm true substructure isomorphism with CDK pattern matching.
when_to_use_negative:
- When the query is a full-structure (identity) match or a similarity/LSH search —
  COCONUT implements those as separate endpoints (hashed InChI/SMILES indexes or LSH-based
  similarity per the ChEMBL blog post), not as fingerprint-bit + CDK isomorphism matching.
- When candidate sets are already exhaustively confirmed or the database is small
  enough for a direct CDK isomorphism scan on all molecules — the $allBitsSet pre-filter
  step is redundant in that case.
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_0154
tools:
- name: CDK
  role: Pattern-matching library used to confirm true substructure isomorphism of
    candidate molecules against the query subgraph (Ullmann default; Vento-Foggia
    or depth-first alternatives).
  repo: https://github.com/mSorok/NaturalProductsOnline
- name: MongoDB
  role: Stores each molecule's PubChem fingerprint as BinData and provides the $allBitsSet
    operator used to pre-filter candidates before CDK confirmation.
  repo: https://github.com/mSorok/COCONUT
- name: Kotlin
  role: Back-end language implementing the substructure search endpoint and coordinating
    the CDK pattern-matching step.
  repo: https://github.com/mSorok/NaturalProductsOnline
- name: Java 11
  role: Back-end runtime (with Kotlin) in which CDK pattern matching is executed.
  repo: https://github.com/mSorok/NaturalProductsOnline
- name: Spring
  role: Spring data framework used to adapt the search logic in the Kotlin/Java back-end.
  repo: https://github.com/mSorok/NaturalProductsOnline
- name: OpenChemLib
  role: Handles the chemical editor for the search functions in the COCONUT web interface.
  repo: https://github.com/mSorok/NaturalProductsOnline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:6132f3ac118af6189c24b1db8fb6ac86fb915122897740d40fc6b0f63544fa53
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
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cdk-substructure-pattern-matching

## Summary

Pattern-matching step of the COCONUT online substructure search: after candidate filtering via $allBitsSet on PubChem fingerprints, CDK confirmatory subgraph isomorphism is run (Ullmann by default, or Vento-Foggia or depth-first) and confirmed matches are written to substructure_search_results.json.

## When to use

Use when a query molecule's subgraph fingerprint has already narrowed candidates (e.g. via $allBitsSet on PubChem fingerprints stored as BinData in MongoDB) and you must confirm true substructure isomorphism with CDK pattern matching.

## When NOT to use

- When the query is a full-structure (identity) match or a similarity/LSH search — COCONUT implements those as separate endpoints (hashed InChI/SMILES indexes or LSH-based similarity per the ChEMBL blog post), not as fingerprint-bit + CDK isomorphism matching.
- When candidate sets are already exhaustively confirmed or the database is small enough for a direct CDK isomorphism scan on all molecules — the $allBitsSet pre-filter step is redundant in that case.

## Inputs

- query molecule subgraph fingerprint (list of PubChem fingerprint bits)
- candidate molecule records from MongoDB whose BinData PubChem fingerprint passed the $allBitsSet filter (the uniqueNaturalProduct collection in COCONUT_2021_03 / COCONUT_2020-07)
- CDK toolkit available in the Kotlin/Java 11 back-end

## Outputs

- confirmed substructure matches, each with molecule identifier, written to substructure_search_results.json
- application/json response from the COCONUT web search endpoint

## How to apply

Given the candidate list produced by MongoDB's $allBitsSet filter on PubChem fingerprints (stored as BinData byte lists in the uniqueNaturalProduct collection), retrieve each candidate and run a CDK pattern-matching algorithm against the query subgraph — the default is Ullmann, with Vento-Foggia or depth-first available as alternatives in the COCONUT back-end (Kotlin/Java 11, Spring). Only molecules whose candidate subgraph is confirmed as a true substructure isomorph are written as confirmed substructure matches, together with molecule identifiers, into substructure_search_results.json, returned as application/json. The two-stage design (fingerprint pre-filter then confirmatory CDK isomorphism check) is the COCONUT approach to substructure search on top of MongoDB, since chemical search is uncommon in MongoDB. Candidate filtering is coarse; every confirmed match must pass the CDK isomorphism check, so false positives from bit overlap are eliminated here, not in MongoDB.

## Related tools

- **CDK** (Pattern-matching library used to confirm true substructure isomorphism of candidate molecules against the query subgraph (Ullmann default; Vento-Foggia or depth-first alternatives).) — https://github.com/mSorok/NaturalProductsOnline
- **MongoDB** (Stores each molecule's PubChem fingerprint as BinData and provides the $allBitsSet operator used to pre-filter candidates before CDK confirmation.) — https://github.com/mSorok/COCONUT
- **Kotlin** (Back-end language implementing the substructure search endpoint and coordinating the CDK pattern-matching step.) — https://github.com/mSorok/NaturalProductsOnline
- **Java 11** (Back-end runtime (with Kotlin) in which CDK pattern matching is executed.) — https://github.com/mSorok/NaturalProductsOnline
- **Spring** (Spring data framework used to adapt the search logic in the Kotlin/Java back-end.) — https://github.com/mSorok/NaturalProductsOnline
- **OpenChemLib** (Handles the chemical editor for the search functions in the COCONUT web interface.) — https://github.com/mSorok/NaturalProductsOnline

## Examples

```
mongorestore --port 27022 --db=COCONUT_2021_03 --noIndexRestore . && mongosh --port 27022 --eval 'db.uniqueNaturalProduct.find({ pubchemBits: { $allBitsSet: queryBits } })' > candidates.json
```

## Evaluation signals

- Every confirmed match passes the CDK pattern-matching check against the query subgraph (no candidate that fails isomorphism is included).
- Results are returned in application/json format.
- Each result carries molecule identifiers so downstream consumers can look up the original COCONUT entry.
- The candidate set entering CDK matching equals the set retained by $allBitsSet on PubChem fingerprint bits — no candidate bypasses the fingerprint pre-filter.
- Confirmed-match count is consistent with the number of candidates that pass CDK isomorphism (no silent truncation or duplicate identifiers).

## Limitations

- False positives from bit-overlap in the PubChem fingerprint pre-filter are handled by the CDK confirmatory step; the article does not quantify the pre-filter's recall/precision trade-off.
- The depth-first and Vento-Foggia alternatives to Ullmann are mentioned as available options but the article does not benchmark them, so choice of algorithm must be made without published guidance.
- Substructure search relies on precomputed PubChem fingerprints being up to date; if fingerprints are stale, $allBitsSet may miss candidates before CDK confirmation even runs.
- Substructure search is specific to the COCONUT back-end; applying it outside the provided Spring/Kotlin/MongoDB stack requires reimplementing the $allBitsSet pre-filter and CDK isomorphism logic.

## Evidence

- [intro] Substructure search pattern-matching step in COCONUT online: "their fingerprints of choice (in COCONUT are used the PubChem fingerprints) to be precomputed and stored as a list of bytes (BinData type in MongoDB)."
- [intro] Confirmatory CDK isomorphism check: "The CDK [13] library is used to process chemical information and formats."
- [intro] Pattern-matching algorithm choice: [evidence span withheld — anchor status: not_found]
- [intro] Spring / Kotlin back-end: "The COCONUT back-end, allowing to process the front-end requests and to communicate with the database is written in Kotlin and Java 11 using the Spring framework."
- [intro] JSON response format: [evidence span withheld — anchor status: not_found]
- [readme] Fingerprint pre-filter in MongoDB: [evidence span withheld — anchor status: external_doc]
