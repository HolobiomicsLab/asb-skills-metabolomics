---
name: graph-isomorphism-search-confirmation
description: Use when a fingerprint/LSH prefilter yields candidate structures and
  you must verify actual subgraph isomorphism before reporting substructure search
  results.
when_to_use_negative:
- You only need a similarity-ranked search (LSH-based similarity in MongoDB) rather
  than an exact subgraph containment test — use the similarity search path instead
- The query is a full-molecule identity lookup (InChI key or hashed InChI index) rather
  than a substructure pattern
- No fingerprint prefilter is available for the target structure collection, making
  the CDK confirmation step the only filter at intractable cost
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0091
tools:
- name: CDK
  role: Runs the pattern-matching algorithms (Ullmann default, Vento-Foggia, depth-first)
    that confirm true substructure matches among fingerprint-prefiltered candidates
- name: MongoDB
  role: Stores precomputed PubChem fingerprints as BinData and provides the $allBitsSet
    query that yields the candidate set before confirmation
- name: Kotlin
  role: Back-end language used with Java 11 and Spring to process front-end search
    requests and orchestrate the fingerprint filter plus CDK confirmation
- name: OpenChemLib
  role: Handles the chemical editor for input of query structures into the search
    functions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:cd7a97793bbf6bbd7dba24ffb865cdcd737b8108285db4944217fe0524372708
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# graph-isomorphism-search-confirmation

## Summary

Confirm that molecules retrieved by a bitset-based prefilter (e.g. MongoDB $allBitsSet on precomputed PubChem fingerprints) are true substructure matches by running CDK pattern-matching graph algorithms, with Ullmann as the default and Vento-Foggia or depth-first search as alternatives.

## When to use

Use when a fingerprint/LSH prefilter yields candidate structures and you must verify actual subgraph isomorphism before reporting substructure search results.

## When NOT to use

- You only need a similarity-ranked search (LSH-based similarity in MongoDB) rather than an exact subgraph containment test — use the similarity search path instead
- The query is a full-molecule identity lookup (InChI key or hashed InChI index) rather than a substructure pattern
- No fingerprint prefilter is available for the target structure collection, making the CDK confirmation step the only filter at intractable cost

## Inputs

- Query molecule substructure (SMILES/MOL or subgraph) with computed PubChem fingerprint bits
- Candidate molecule set retrieved from MongoDB uniqueNaturalProduct collection via $allBitsSet on pubchemBits BinData fields
- CDK toolkit dependency (Ullmann, Vento-Foggia, or depth-first pattern-matching implementations)

## Outputs

- substructure_search_results.json containing confirmed substructure matches with molecule identifiers (application/json)
- Set of molecule IDs whose structures truly contain the query subgraph per CDK pattern matching

## How to apply

First prefilter the molecule collection with a subgraph fingerprint: COCONUT stores PubChem fingerprints as BinData in MongoDB and matches the query's bits using the native $allBitsSet function, retaining only molecules whose fingerprint has all query bits set. Then treat the candidate set as graph-isomorphism candidates and run CDK pattern matching to eliminate fingerprint false positives. The default algorithm is Ullmann; Vento-Foggia and a depth-first search alternative are available, so choose the algorithm per performance/precision trade-off for the query graph. Persist confirmed hits — molecule identifiers plus substructure match status — as application/json (e.g. substructure_search_results.json). Rationale: fingerprint bits are necessary but not sufficient for subgraph isomorphism, so a cheminformatics pattern matcher on the actual structure is required to avoid spurious matches.

## Related tools

- **CDK** (Runs the pattern-matching algorithms (Ullmann default, Vento-Foggia, depth-first) that confirm true substructure matches among fingerprint-prefiltered candidates)
- **MongoDB** (Stores precomputed PubChem fingerprints as BinData and provides the $allBitsSet query that yields the candidate set before confirmation)
- **Kotlin** (Back-end language used with Java 11 and Spring to process front-end search requests and orchestrate the fingerprint filter plus CDK confirmation)
- **OpenChemLib** (Handles the chemical editor for input of query structures into the search functions)

## Examples

```
// Kotlin/Spring back-end (COCONUT online):
db.uniqueNaturalProduct.find({ pubchemBits: { $allBitsSet: queryBits } }) // candidates -> cdk PatternMatcher.ullmann(subgraph).matchAll(molecule)
```

## Evaluation signals

- Every reported match in substructure_search_results.json passes the CDK pattern-matching check against the query subgraph (no fingerprint-only hits survive)
- Results are served/returned in application/json format
- The candidate set entering CDK confirmation equals the set returned by the MongoDB $allBitsSet query on the query's PubChem fingerprint bits
- Confirmed hits contain valid molecule identifiers (coconut_id / originalInchiKey) present in the uniqueNaturalProduct collection
- Benchmarking against a known substructure-annotated subset shows zero false positives in confirmed results (precision = 1.0); recall is bounded by the $allBitsSet prefilter

## Limitations

- Fingerprint prefiltering is necessary but not sufficient: molecules with all query bits set can still be false positives, so pattern matching is mandatory
- The result set is bounded by the fingerprint prefilter — true substructure matches whose PubChem fingerprint bits do not fully contain the query bits cannot be recovered
- CDK pattern matching on large candidate sets is the performance bottleneck; algorithm choice (Ullmann vs Vento-Foggia vs depth-first) trades speed against matching behavior
- Query structures must be chemical-editor-valid (OpenChemLib) and parseable by CDK; structures failing curation pipelines (e.g. ChEMBL error code ≥ 6) were already removed from COCONUT

## Evidence

- [intro] fingerprint prefilter with $allBitsSet: "their fingerprints of choice (in COCONUT are used the PubChem fingerprints) to be precomputed and stored as a list of bytes (BinData type in MongoDB)."
- [intro] CDK pattern-matching confirmation: "The CDK [13] library is used to process chemical information and formats."
- [intro] database role: "All original data, unified NPs and the derived and calculated information are stored in MongoDB."
- [intro] backend stack for search orchestration: "The COCONUT back-end, allowing to process the front-end requests and to communicate with the database is written in Kotlin and Java 11"
- [readme] indexed fingerprint field in DB: [evidence span withheld — anchor status: external_doc]
