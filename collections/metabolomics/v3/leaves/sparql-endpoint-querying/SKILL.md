---
name: sparql-endpoint-querying
description: Use when a research question about natural product occurrences (structure-organism
  pairs, their references, or chemical/organism metadata) can be answered by querying
  LOTUS data hosted on Wikidata via SPARQL.
when_to_use_negative:
- When the goal is structural or spectral search over natural products — Wikidata
  does not support these query modes; use the parallel LOTUS hosting at lotus.naturalproducts.net
  instead.
- When only the latest curated LOTUS release file is needed — downloading the Zenodo
  release (e.g. doi 10.5281/zenodo.5794597) may be simpler than live SPARQL querying.
- When querying large non-Wikidata RDF graphs — this skill is grounded in the Wikidata
  endpoint and its LOTUS statements; other endpoints may have different schema and
  limits.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3336
tools:
- name: SPARQL
  role: Query language used against the Wikidata SPARQL endpoint to retrieve LOTUS
    structure-organism pairs and provenance references
- name: lotus-wikidata-interact
  role: Programs for uploading and downloading LOTUS data to/from Wikidata, complementing
    direct SPARQL querying
  repo: https://github.com/lotusnprod/lotus-wikidata-interact
- name: lotus-web
  role: Parallel LOTUS frontend providing structural search and exports that Wikidata
    SPARQL does not support
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:8474a896a1dfc4f5e52bf25618359dc471368a296f7cf4bf8c67e66de570b706
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-wikidata-interact
  tools:
  - SPARQL
  - lotus-web
  - lotus-wikidata-interact
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# sparql-endpoint-querying

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Query the public Wikidata SPARQL endpoint to retrieve LOTUS referenced structure-organism pairs and their provenance, enabling ad-hoc exploration of the 750,000+ pair knowledge base without downloading the full dataset.

## When to use

Use when a research question about natural product occurrences (structure-organism pairs, their references, or chemical/organism metadata) can be answered by querying LOTUS data hosted on Wikidata via SPARQL.

## When NOT to use

- When the goal is structural or spectral search over natural products — Wikidata does not support these query modes; use the parallel LOTUS hosting at lotus.naturalproducts.net instead.
- When only the latest curated LOTUS release file is needed — downloading the Zenodo release (e.g. doi 10.5281/zenodo.5794597) may be simpler than live SPARQL querying.
- When querying large non-Wikidata RDF graphs — this skill is grounded in the Wikidata endpoint and its LOTUS statements; other endpoints may have different schema and limits.

## Inputs

- Wikidata SPARQL endpoint URL (https://query.wikidata.org)
- SPARQL query text selecting structure-organism pairs with LOTUS provenance qualifiers
- SPARQL client (e.g. SPARQLWrapper, rdflib) configured with a User-Agent header

## Outputs

- JSON file (e.g. wikidata_lotus_pairs_count.json) with retrieved triples/counts and execution metadata
- Set of structure-organism pair records with references (DOI/PMID where available)

## How to apply

Connect to the Wikidata SPARQL endpoint at https://query.wikidata.org using a SPARQL client such as SPARQLWrapper or rdflib in Python, sending a descriptive User-Agent as the endpoint requires. Formulate a SPARQL query that selects statements linking a chemical structure item to a biological organism item whose provenance reference (stated-in qualifier) points to the LOTUS initiative — the article reports that such a query for all pairs returned 951,800 entries. Paginate large result sets to avoid endpoint timeouts, and aggregate basic metadata (query text, execution timestamp, endpoint) alongside the results. Note that Wikidata lacks structural and spectral search modes, so the SPARQL route is best for graph/provenance queries rather than structure search; for the latter use the parallel hosting on lotus.naturalproducts.net or the lotus-web frontend (docker-compose build; docker-compose up -d).

## Related tools

- **SPARQL** (Query language used against the Wikidata SPARQL endpoint to retrieve LOTUS structure-organism pairs and provenance references)
- **lotus-wikidata-interact** (Programs for uploading and downloading LOTUS data to/from Wikidata, complementing direct SPARQL querying) — https://github.com/lotusnprod/lotus-wikidata-interact
- **lotus-web** (Parallel LOTUS frontend providing structural search and exports that Wikidata SPARQL does not support)

## Examples

```
from SPARQLWrapper import SPARQLWrapper, JSON; sp = SPARQLWrapper('https://query.wikidata.org/sparql'); sp.setQuery('SELECT ?s ?o WHERE { ?s wdt:P710 ?o . ?st ps:P710 ?o; pq:P248 wd:Q105908578 }'); sp.setReturnFormat(JSON); r = sp.query().convert()
```

## Evaluation signals

- The results file exists and parses as valid JSON
- It contains a nonzero triple/pair count field (article reports ~951,800 pairs for an all-pairs query)
- The same query re-executed on the endpoint reproduces the same count (reproducibility check)
- Execution metadata (endpoint, timestamp, query text) is present alongside the count
- Every returned structure-organism pair carries a LOTUS provenance qualifier in its statements

## Limitations

- Wikidata does not offer structural or spectral search modes typical of molecular NP resources; these shortcomings motivated the parallel hosting on lotus.naturalproducts.net
- The dataset mirrors may lag behind community edits made directly on Wikidata (regular mirroring to LNPN)
- SPARQL queries over ~1M statements can hit endpoint timeouts without pagination
- Manual entry of referenced structure-organism pairs into Wikidata is possible but error-prone without the automatic filtering the LOTUS pipeline applies

## Evidence

- [full_text] SPARQL client usage pattern: [evidence span withheld — anchor status: not_found]
- [full_text] Reproducible pair-count validation: [evidence span withheld — anchor status: not_found]
- [readme] lotus-web provides structural search: [evidence span withheld — anchor status: external_doc]
