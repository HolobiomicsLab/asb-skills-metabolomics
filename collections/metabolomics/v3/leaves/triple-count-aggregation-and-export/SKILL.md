---
name: triple-count-aggregation-and-export
description: Use when you need a quantitative coverage figure for a Wikidata-hosted
  dataset — e.g. reporting how many structure-organism pairs LOTUS contains via the
  public SPARQL endpoint at query.wikidata.org.
when_to_use_negative:
- The target dataset is not hosted on or queryable via a SPARQL endpoint — e.g. raw
  RDF dumps or flat CSV tables where SPARQL counting is unnecessary indirection
- You need full records for downstream structural or spectral search rather than a
  summary count — the article notes Wikidata lacks structural/spectral search modes,
  so LOTUS's parallel host (lotus.naturalproducts.net) is the better path
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3489
tools:
- name: SPARQL
  role: Query language used against the public Wikidata SPARQL endpoint to retrieve
    and count LOTUS-referenced structure-organism pairs
- name: lotusnprod/lotus-processor
  role: Upstream pipeline that harmonizes, curates, validates, and prepares the LOTUS
    structure-organism pairs that are later counted on Wikidata
  repo: https://github.com/lotusnprod/lotus-processor
- name: lotusnprod/lotus-wikidata-interact
  role: Contains uploadLotus (uploads LOTUS data to Wikidata) and downloadLotus (downloads
    LOTUS data from Wikidata, also used to check what already exists) — the programmatic
    counterpart to SPARQL-based retrieval
  repo: https://github.com/lotusnprod/lotus-wikidata-interact
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:50181000dfee49970223cc19a025147342d22e94855b7cb41921b5074811ca17
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-processor
  tools:
  - SPARQL
  - lotusnprod/lotus-processor
  - lotusnprod/lotus-wikidata-interact
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# triple-count-aggregation-and-export

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Count all Wikidata statements linking a chemical structure to a biological organism whose provenance references the LOTUS initiative, then record the count with reproducible query metadata and export it as a JSON artifact. This turns a large, queryable knowledge base into a single auditable coverage metric for the LOTUS dataset.

## When to use

Use when you need a quantitative coverage figure for a Wikidata-hosted dataset — e.g. reporting how many structure-organism pairs LOTUS contains via the public SPARQL endpoint at query.wikidata.org.

## When NOT to use

- The target dataset is not hosted on or queryable via a SPARQL endpoint — e.g. raw RDF dumps or flat CSV tables where SPARQL counting is unnecessary indirection
- You need full records for downstream structural or spectral search rather than a summary count — the article notes Wikidata lacks structural/spectral search modes, so LOTUS's parallel host (lotus.naturalproducts.net) is the better path

## Inputs

- Wikidata SPARQL endpoint (https://query.wikidata.org)
- SPARQL query selecting chemical-structure-to-organism statements with LOTUS provenance (stated-in qualifier)
- SPARQL client (SPARQLWrapper or rdflib) with a descriptive User-Agent string

## Outputs

- Structure-organism triples retrieved from the endpoint
- Count of retrieved triples (e.g. 951,800 for the LOTUS demonstration query)
- Metadata record: query text, execution timestamp, endpoint
- Exported JSON file, e.g. wikidata_lotus_pairs_count.json

## How to apply

Connect to the Wikidata SPARQL endpoint (https://query.wikidata.org) with a Python SPARQL client such as SPARQLWrapper or rdflib, sending a descriptive User-Agent as the endpoint requires. Formulate a SPARQL query that selects statements linking a chemical structure item to a biological organism item whose provenance reference (stated-in qualifier) points to the LOTUS initiative — in the article's own demonstration such a query returned 951,800 structure-organism pairs. Execute the query, paginating to avoid endpoint timeouts if the result set is large, and collect the resulting triples. Aggregate metadata alongside the count: the query text, execution timestamp, and endpoint used. Export the count and metadata to a JSON file (e.g. wikidata_lotus_pairs_count.json) so the metric can be validated and reproduced by re-running the same query.

## Related tools

- **SPARQL** (Query language used against the public Wikidata SPARQL endpoint to retrieve and count LOTUS-referenced structure-organism pairs)
- **lotusnprod/lotus-processor** (Upstream pipeline that harmonizes, curates, validates, and prepares the LOTUS structure-organism pairs that are later counted on Wikidata) — https://github.com/lotusnprod/lotus-processor
- **lotusnprod/lotus-wikidata-interact** (Contains uploadLotus (uploads LOTUS data to Wikidata) and downloadLotus (downloads LOTUS data from Wikidata, also used to check what already exists) — the programmatic counterpart to SPARQL-based retrieval) — https://github.com/lotusnprod/lotus-wikidata-interact

## Examples

```
from SPARQLWrapper import SPARQLWrapper, JSON; sparql = SPARQLWrapper('https://query.wikidata.org/sparql'); sparql.setQuery('SELECT ?structure ?organism ?ref WHERE {?structure wdt:P703 ?organism. ?structure p:P703 ?statement. ?statement pq:P248 wd:Q104858806. ?statement prov:wasDerivedFrom ?ref.}'); sparql.setReturnFormat(JSON); results = sparql.query().convert()
```

## Evaluation signals

- The results file exists and parses as valid JSON
- The JSON contains a nonzero triple-count field
- Re-executing the same SPARQL query reproduces the same count (reproducibility check)
- The exported metadata records the query text, execution timestamp, and endpoint
- The count is consistent with the article's reported scale (951,800 pairs in the demonstration query)

## Limitations

- Very large result sets may require pagination to avoid Wikidata endpoint timeouts
- The count reflects only pairs whose provenance references LOTUS — other LOTUS-related statements (or statements with different qualifiers) are excluded
- The LOTUS Wikidata interactions repository is archived; development continues at github.com/lotus-initiative, so endpoint/qualifier conventions may evolve
- Wikidata alone does not support structural or spectral search, which is why LOTUS is mirrored in parallel at https://lotus.naturalproducts.net

## Evidence

- [results] 951,800 pairs returned by demonstration query: [evidence span withheld — anchor status: not_found]
- [other] User-Agent requirement on Wikidata SPARQL endpoint: [evidence span withheld — anchor status: not_found]
