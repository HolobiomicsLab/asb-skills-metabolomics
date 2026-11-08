---
name: wikidata-structure-organism-triple-retrieval
description: Use when a task requires enumerating or counting LOTUS referenced structure-organism
  pairs (chemical structure item linked to biological organism item) via the public
  Wikidata SPARQL endpoint at https://query.wikidata.org.
when_to_use_negative:
- When structural or spectral searches are needed — these query modes are not yet
  available in Wikidata; use the parallel mirror at https://lotus.naturalproducts.net
  instead
- When a full bulk download of the LOTUS dataset is required — paginated SPARQL retrieval
  is inefficient; prefer the downloadLotus tooling or Zenodo archives (e.g. DOI 10.5281/zenodo.5794597)
- When the goal is community curation/upsert of new pairs into Wikidata rather than
  retrieval — use the upload path (manual editing or upsert scripts in lotus-wikidata-interact)
  instead
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: SPARQL
  role: Query language used against the Wikidata knowledge base to explore the dataset
    and retrieve structure-organism pairs with LOTUS provenance
- name: SPARQLWrapper / rdflib
  role: Python SPARQL clients for connecting to https://query.wikidata.org and executing
    the paginated structure-organism pair query
- name: lotus-wikidata-interact (downloadLotus)
  role: Companion program used to download LOTUS data from Wikidata and to check which
    entries already exist on Wikidata
  repo: https://github.com/lotus-initiative
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:c32976c79dda6a6a8a0d5480485606fb92f96346f105500c3e46708eb2aaa8c4
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - SPARQL
  - SPARQLWrapper / rdflib
  - lotus-wikidata-interact (downloadLotus)
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# wikidata-structure-organism-triple-retrieval

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Retrieves the LOTUS-initiative referenced structure-organism pairs mirrored on Wikidata by issuing SPARQL queries against the public endpoint at https://query.wikidata.org, and exports the resulting triple count with provenance metadata. LOTUS holds 750,000+ curated pairs, and a full-pair query returned 951,800 entries.

## When to use

Use when a task requires enumerating or counting LOTUS referenced structure-organism pairs (chemical structure item linked to biological organism item) via the public Wikidata SPARQL endpoint at https://query.wikidata.org.

## When NOT to use

- When structural or spectral searches are needed — these query modes are not yet available in Wikidata; use the parallel mirror at https://lotus.naturalproducts.net instead
- When a full bulk download of the LOTUS dataset is required — paginated SPARQL retrieval is inefficient; prefer the downloadLotus tooling or Zenodo archives (e.g. DOI 10.5281/zenodo.5794597)
- When the goal is community curation/upsert of new pairs into Wikidata rather than retrieval — use the upload path (manual editing or upsert scripts in lotus-wikidata-interact) instead

## Inputs

- SPARQL query string linking a Wikidata chemical structure item to a biological organism item with a stated-in (P248) reference qualifier to LOTUS
- Wikidata SPARQL endpoint URL (https://query.wikidata.org)
- Pagination window size (LIMIT/OFFSET) to avoid endpoint timeouts

## Outputs

- List of structure-organism triples (chemical structure item, biological organism item) with LOTUS provenance references
- wikidata_lotus_pairs_count.json containing the triple count plus metadata (query text, execution timestamp, endpoint)

## How to apply

Connect to https://query.wikidata.org with a SPARQL client such as SPARQLWrapper or rdflib in Python, sending a descriptive User-Agent as the endpoint requires. Formulate a SPARQL query selecting statements that link a chemical structure item to a biological organism item whose provenance reference (the stated-in qualifier, P248) points to the LOTUS initiative, e.g. via the ?structure wdt:P703 ?organism pattern. Execute the query and paginate (LIMIT/OFFSET) if necessary to avoid endpoint timeouts — a query for all pairs returned 951,800 entries, far beyond a single response window. Count the retrieved triples and aggregate basic metadata (query text, execution timestamp, endpoint) alongside the count. Export the count and metadata to a JSON file such as wikidata_lotus_pairs_count.json. Validate that the file exists, parses as JSON, contains a nonzero triple count field, and that the count is reproducible on re-execution of the same query; note that the curated validated subset in the article is 750,000+ pairs (~30% of 2.5M+ initial pairs), so raw-query counts will be higher.

## Related tools

- **SPARQL** (Query language used against the Wikidata knowledge base to explore the dataset and retrieve structure-organism pairs with LOTUS provenance)
- **SPARQLWrapper / rdflib** (Python SPARQL clients for connecting to https://query.wikidata.org and executing the paginated structure-organism pair query)
- **lotus-wikidata-interact (downloadLotus)** (Companion program used to download LOTUS data from Wikidata and to check which entries already exist on Wikidata) — https://github.com/lotus-initiative

## Examples

```
from SPARQLWrapper import SPARQLWrapper; q = SPARQLWrapper("https://query.wikidata.org/sparql"); q.setQuery("SELECT ?structure ?organism WHERE { ?structure wdt:P703 ?organism } LIMIT 10000 OFFSET 0"); r = q.query().convert()
```

## Evaluation signals

- wikidata_lotus_pairs_count.json exists and parses as valid JSON
- The results file contains a nonzero triple count field
- The count is reproducible on re-execution of the same SPARQL query
- Retrieved count is in a plausible range: a full-pair query returned 951,800 entries, while the curated validated subset reported in the paper is 750,000+ (~30% of 2.5M+ initial pairs)
- The HTTP client sends a descriptive User-Agent as required by the Wikidata endpoint

## Limitations

- Wikidata does not yet offer typical NP-resource query modes such as structural or spectral searches; the LOTUS website mirror exists precisely to address these shortcomings
- Large all-pair queries must be paginated (LIMIT/OFFSET) or they will hit endpoint timeouts
- Wikidata is community-editable, so pair counts drift over time and may differ between the Wikidata version and the lotus.naturalproducts.net mirror

## Evidence

- [readme] downloadLotus retrieves LOTUS data from Wikidata: [evidence span withheld — anchor status: external_doc]
