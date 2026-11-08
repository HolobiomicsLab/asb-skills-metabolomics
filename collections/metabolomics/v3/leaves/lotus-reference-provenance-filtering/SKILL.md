---
name: lotus-reference-provenance-filtering
description: Use when you need to extract only referenced structure–organism pairs
  attributable to the LOTUS initiative from Wikidata, or reproduce LOTUS' reference/provenance
  validation filters on candidate occurrence pairs.
when_to_use_negative:
- You need structural or spectral search over natural products data — these query
  modes are not available in Wikidata and require the mirrored LOTUS site (lotus.naturalproducts.net)
  instead
- Candidate pairs carry no reference metadata at all (no DOI, PMID, or comparable
  title), so reference-level provenance validation cannot decide validity
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SPARQL
  role: Query language used against the Wikidata knowledge base to select structure–organism
    statements filtered by the LOTUS stated-in reference qualifier
- name: lotus-wikidata-interact
  role: Provides downloadLotus, used to download LOTUS data from Wikidata and to check
    what already exists on Wikidata before upload/upsert
  repo: https://github.com/lotusnprod/lotus-wikidata-interact
- name: lotus-processor
  role: Contains the curation scripts (e.g. filter_dirty.R) implementing the automatic
    reference/entry filtering applied to all LOTUS entries
  repo: https://github.com/lotusnprod/lotus-processor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:0731002ea82f95810162c93b36454eb7cfa8117b0ceeb2ae36305a1f7c56d277
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-wikidata-interact
  tools:
  - SPARQL
  - lotus-processor
  - lotus-wikidata-interact
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# lotus-reference-provenance-filtering

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Retrieves and filters only LOTUS-provenance-backed referenced structure–organism pairs, either by querying Wikidata SPARQL statements whose stated-in reference qualifier points to the LOTUS initiative, or by applying LOTUS' reference-level validation rules (DOI/PMID, Damerau-Levenshtein distance <10, two-of-three publishing details).

## When to use

Use when you need to extract only referenced structure–organism pairs attributable to the LOTUS initiative from Wikidata, or reproduce LOTUS' reference/provenance validation filters on candidate occurrence pairs.

## When NOT to use

- You need structural or spectral search over natural products data — these query modes are not available in Wikidata and require the mirrored LOTUS site (lotus.naturalproducts.net) instead
- Candidate pairs carry no reference metadata at all (no DOI, PMID, or comparable title), so reference-level provenance validation cannot decide validity

## Inputs

- Connection to the Wikidata SPARQL endpoint (https://query.wikidata.org)
- SPARQL query text selecting structure–organism statements with a stated-in reference qualifier pointing to the LOTUS initiative
- Optional local candidate structure–organism pair records with reference metadata (DOI, PMID, title, year, journal, first author)

## Outputs

- List of LOTUS-referenced structure–organism triples (a full query returned 951,800 entries)
- Count and provenance metadata exported as JSON (e.g. wikidata_lotus_pairs_count.json containing query text, execution timestamp, endpoint, and triple count)

## How to apply

Connect to the public Wikidata SPARQL endpoint (https://query.wikidata.org) with a SPARQL client such as SPARQLWrapper or rdflib in Python, sending a descriptive User-Agent as the endpoint requires. Formulate a SPARQL query selecting statements that link a chemical structure item to a biological organism item whose provenance reference (the stated-in qualifier on the reference) points to the LOTUS initiative; execute with pagination to avoid endpoint timeouts, then count the retrieved triples and aggregate metadata (query text, execution timestamp, endpoint) alongside the count in a JSON export. When filtering candidate pairs offline instead, apply LOTUS' reference validation: accept entries whose reference is a DOI, a PMID, or whose restricted Damerau-Levenshtein distance between original and translated title is lower than ten, and keep publishingDetails only if at least two of year, journal and first author are present. The rationale is provenance grounding: LOTUS retained roughly 30% of 2.5M+ initial pairs (750,000+) only after manual and automatic validation, so unfiltered pairs include invalid occurrences.

## Related tools

- **SPARQL** (Query language used against the Wikidata knowledge base to select structure–organism statements filtered by the LOTUS stated-in reference qualifier)
- **lotus-wikidata-interact** (Provides downloadLotus, used to download LOTUS data from Wikidata and to check what already exists on Wikidata before upload/upsert) — https://github.com/lotusnprod/lotus-wikidata-interact
- **lotus-processor** (Contains the curation scripts (e.g. filter_dirty.R) implementing the automatic reference/entry filtering applied to all LOTUS entries) — https://github.com/lotusnprod/lotus-processor

## Examples

```
from SPARQLWrapper import SPARQLWrapper; s=SPARQLWrapper('https://query.wikidata.org/sparql'); s.setQuery('SELECT ?structure ?organism WHERE { ?structure p:P703 ?st . ?st prov:wasDerivedFrom/pr:P248 ?lotusRef }')
```

## Evaluation signals

- The exported results file exists, parses as JSON, and contains a nonzero triple-count field
- The triple count is reproducible on re-execution of the same query (e.g. on the order of 951,800 pairs for all LOTUS structure–organism statements)
- Every retained pair carries a provenance reference (stated-in qualifier) pointing to the LOTUS initiative
- Offline filtering reproduces the article's retention invariant: roughly 30% of 2.5M+ initial pairs survive validation (750,000+ retained)
- Reference filters behave as specified: DOI/PMID references accepted, translated titles accepted only at Damerau-Levenshtein distance <10, publishing details kept only with ≥2 of year/journal/first author

## Limitations

- The Wikidata SPARQL endpoint can time out on unpaginated bulk queries; large extractions must be paginated
- The LOTUS processing pipeline is heavy, includes many dependencies and is convoluted, making local reproduction of the filtering non-trivial
- The current processing pipeline still removes correct entries and misses or induces incorrect ones, and no exact count of these errors is given
- Wikidata is community-editable, so retrieved counts can drift over time between query executions

## Evidence

- [readme] downloadLotus role: [evidence span withheld — anchor status: external_doc]
