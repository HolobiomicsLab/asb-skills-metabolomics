---
name: paired-omics-project-filtering
description: Use when the analysis goal is to find PoDP projects by extraction or
  sample-preparation metadata (e.g. projects that used methanol to extract cell pellets)
  via keyword search over project documents.
when_to_use_negative:
- The target metadata was entered as free text in the platform's 'Other' boxes — it
  is inherently not machine-readable, so keyword matching over structured fields will
  be unreliable.
- The goal is reliably selecting datasets by a controlled metadata criterion (e.g.
  only metagenomic datasets from aquatic invertebrates or only positive ionization
  mode) — use the platform's ontology-based selection rather than free-text keyword
  search.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Paired Omics Data Platform
  role: Provides the List-tab keyword search (Elasticsearch full-text index) over
    project metadata and the project JSON schema used for output
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Hosts the regularly archived downloadable JSON dump of all PoDP projects used
    as the searchable corpus
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:489fae1545739227d78968a4f657008de9b6cf730356384c1b598ec6390a60f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# paired-omics-project-filtering

## Summary

A keyword-search filtering skill for the Paired Omics Data Platform (PoDP): retrieve projects whose modular extraction-method and sample-preparation metadata match a query such as "methanol + cells", by searching the platform's List-tab index over the Zenodo-archived JSON project documents.

## When to use

Use when the analysis goal is to find PoDP projects by extraction or sample-preparation metadata (e.g. projects that used methanol to extract cell pellets) via keyword search over project documents.

## When NOT to use

- The target metadata was entered as free text in the platform's 'Other' boxes — it is inherently not machine-readable, so keyword matching over structured fields will be unreliable.
- The goal is reliably selecting datasets by a controlled metadata criterion (e.g. only metagenomic datasets from aquatic invertebrates or only positive ionization mode) — use the platform's ontology-based selection rather than free-text keyword search.

## Inputs

- PoDP project documents in Zenodo-archived JSON format (modular extraction method and sample preparation metadata fields)
- Free-text keyword query, e.g. 'methanol + cells'

## Outputs

- Filtered list of PoDP projects whose extraction/sample-preparation fields contain the query terms (e.g. methanol_cells_projects.json matching the platform's project document structure)

## How to apply

Load the PoDP project documents as archived to Zenodo in JSON format, each containing modular sample preparation, extraction method and instrumentation method metadata fields. Execute the platform's free-text search semantics — the web application offers full-text search via an Elasticsearch (v7.6.2) index exposed through the List tab — with the query terms (e.g. 'methanol' and 'cells') over the extraction/sample-preparation text. Collect the returned projects and, for each, inspect its extraction-method and sample-preparation fields to confirm the queried terms are present (substring presence of 'methanol' and 'cells'). Retain only projects whose fields verify the claim and exclude search hits that do not contain the terms; write the verified matches with their extraction metadata as JSON matching the platform's project document structure (schema.json).

## Related tools

- **Paired Omics Data Platform** (Provides the List-tab keyword search (Elasticsearch full-text index) over project metadata and the project JSON schema used for output) — https://github.com/iomega/paired-data-form
- **Zenodo** (Hosts the regularly archived downloadable JSON dump of all PoDP projects used as the searchable corpus)

## Examples

```
python -c "import json; d=json.load(open('pods.json')); print([p['id'] for p in d if 'methanol' in json.dumps(p).lower() and 'cells' in json.dumps(p).lower()])"
```

## Evaluation signals

- Every retained project's extraction/sample-preparation metadata fields contain the query terms (substring presence of 'methanol' and 'cells').
- Projects returned by the search whose fields do not contain the query terms are excluded.
- Output JSON conforms to the platform's project document structure (app/public/schema.json).
- The set of retained projects matches what the platform's List-tab query returns for 'methanol + cells' (projects that used methanol to extract cell pellets).

## Limitations

- Free text entered in the 'Other' boxes is inherently not machine-readable, so relevant projects may be missed if the queried metadata lives only in such free-text fields.
- Keyword search is not ontology-aware: capturing the full range of relevant variables in standardized machine-readable format was judged too complex by the platform designers, so exact-term matching is the operative semantics.
- Monthly Zenodo archiving means the searchable JSON corpus may lag the live platform.

## Evidence

- [abstract] Keyword search over extraction metadata: "searching “methanol + cells” retrieves projects that used methanol to extract cell pellets"
- [abstract] List-tab keyword search entry point: "users can search for projects under the “List” tab, using keywords to find studies of interest"
- [abstract] Zenodo JSON archive as corpus: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [readme] Elasticsearch full-text search backend: [evidence span withheld — anchor status: external_doc]
- [abstract] Modular metadata structure: "Minimal metadata with information about sample preparation and data collection are recorded in a modular way"
