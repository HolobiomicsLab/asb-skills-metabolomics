---
name: omics-metadata-keyword-search
description: Use when a research question asks whether keyword search over PoDP project
  metadata (e.g. extraction methods like 'methanol + cells') can retrieve the projects
  whose archived JSON documents match that extraction method.
when_to_use_negative:
- The target metadata is free text entered in the 'Other' boxes, which is inherently
  not machine-readable and cannot be reliably matched by keyword filters.
- The goal is ontology-based dataset selection (e.g. only metagenomic datasets from
  aquatic invertebrates, or only positive ionization mode) rather than simple extraction-method
  keyword retrieval.
- The query targets data stored in MassIVE/MetaboLights/GenBank rather than the PoDP's
  own project metadata — the PoDP does not store metabolomics or genomics datasets,
  only their linking metadata.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0219
tools:
- name: Paired Omics Data Platform
  role: Provides the List-tab keyword search over project metadata and the project
    JSON document structure being filtered
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Source of the regularly archived JSON dataset of community-curated PoDP project
    documents that are loaded and filtered
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:87fac41f1b23562cf03af699b5bf0b9c2fec15abffd4d177d180162bccd8d75b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# omics-metadata-keyword-search

## Summary

Reproduces the PoDP 'methanol + cells' free-text search that retrieves projects by their extraction-method metadata, validating that each returned project's extraction/sample-preparation fields actually contain the queried terms.

## When to use

Use when a research question asks whether keyword search over PoDP project metadata (e.g. extraction methods like 'methanol + cells') can retrieve the projects whose archived JSON documents match that extraction method.

## When NOT to use

- The target metadata is free text entered in the 'Other' boxes, which is inherently not machine-readable and cannot be reliably matched by keyword filters.
- The goal is ontology-based dataset selection (e.g. only metagenomic datasets from aquatic invertebrates, or only positive ionization mode) rather than simple extraction-method keyword retrieval.
- The query targets data stored in MassIVE/MetaboLights/GenBank rather than the PoDP's own project metadata — the PoDP does not store metabolomics or genomics datasets, only their linking metadata.

## Inputs

- Archived Zenodo JSON documents of PoDP projects (project document structure per app/public/schema.json)
- Free-text keyword query 'methanol + cells' run against the PoDP List-tab keyword index

## Outputs

- methanol_cells_projects.json — verified matching projects with their extraction-method and sample-preparation metadata fields

## How to apply

Load the community-curated PoDP project documents archived monthly to Zenodo in JSON format; each project contains modular metadata fields for sample preparation, extraction method, and instrumentation. Execute the platform's List-tab free-text search semantics — backed by an Elasticsearch (v7.6.2) index per the platform README — with the query 'methanol + cells' over the extraction/sample-preparation text. Collect the returned projects and, for each, inspect its extraction-method and sample-preparation fields to confirm substring presence of both 'methanol' and 'cells'. Retain only projects whose fields verify the claim (true positives of the search), excluding returned projects that lack the terms, and write the verified matches with their extraction metadata to methanol_cells_projects.json in the platform's project document structure (as described by app/public/schema.json).

## Related tools

- **Paired Omics Data Platform** (Provides the List-tab keyword search over project metadata and the project JSON document structure being filtered) — https://github.com/iomega/paired-data-form
- **Zenodo** (Source of the regularly archived JSON dataset of community-curated PoDP project documents that are loaded and filtered)

## Evaluation signals

- Every retained project's extraction/sample-preparation metadata fields contain substring 'methanol' AND substring 'cells'.
- Any project returned by the 'methanol + cells' query whose fields lack the query terms is excluded (no false positives in output).
- Output JSON conforms to the platform's project document structure (app/public/schema.json).
- Result set is reproducible: re-running the List-tab keyword search on the same Zenodo archive returns the same candidate projects.
- Output file methanol_cells_projects.json parses as valid JSON and lists matching projects with their extraction metadata.

## Limitations

- Free text entered in the 'Other' boxes is inherently not machine-readable, so keyword search may miss customized extraction descriptions.
- The PoDP does not store the underlying metabolomics or genomics datasets — only linking metadata — so retrieval yields project records, not raw data.
- The article recommends ontologies for reliably selecting datasets by specific metadata; plain keyword search is a coarser filter than ontology-based selection.

## Evidence

- [abstract] searching “methanol + cells” retrieves projects that used methanol to extract cell pellets: "searching “methanol + cells” retrieves projects that used methanol to extract cell pellets"
- [abstract] users can search for projects under the “List” tab, using keywords to find studies of interest: "users can search for projects under the “List” tab, using keywords to find studies of interest"
- [abstract] These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [readme] The application offers full text search functionality via web services using an elastic search (v7.6.2) index.: [evidence span withheld — anchor status: external_doc]
- [abstract] Free text entered in the “Other” boxes is inherently not machine-readable but gives an option for customization by the user: "Free text entered in the “Other” boxes is inherently not machine-readable but gives an option for customization by the user"
