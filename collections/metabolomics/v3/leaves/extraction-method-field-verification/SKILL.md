---
name: extraction-method-field-verification
description: Use when a retrieval claim about PoDP extraction-method metadata (e.g.
  'methanol + cells' finds methanol cell-pellet extractions) must be reproduced and
  field-level verified against archived Zenodo JSON project documents.
when_to_use_negative:
- The metadata value you need to filter on is free text entered in an 'Other' box
  — such text is inherently not machine-readable, so substring verification cannot
  be relied upon
- You need ontology-based, reliable selection by specific metadata (e.g. only metagenomic
  datasets from aquatic invertebrates, or only a specific culture medium) rather than
  free-text keyword matching
- The target records are BGC–MS/MS links (MIBiG identifier + MS/MS URL + scan number)
  rather than project-level extraction/sample-preparation metadata
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: Paired Omics Data Platform
  role: Provides the List-tab keyword search ('methanol + cells') over project extraction/sample-preparation
    text and the project JSON documents whose metadata fields are verified
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Source of the regularly archived, downloadable JSON project dataset used for
    offline field verification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:e210e361978e443522be4768fd9480042680527ca278d630a769ac3c4b4d859a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# extraction-method-field-verification

## Summary

Reproduces and validates the PoDP 'methanol + cells' extraction-method keyword search: projects returned by the platform's List-tab keyword search are verified by confirming substring presence of the query terms ('methanol' and 'cells') in their extraction-method and sample-preparation metadata fields.

## When to use

Use when a retrieval claim about PoDP extraction-method metadata (e.g. 'methanol + cells' finds methanol cell-pellet extractions) must be reproduced and field-level verified against archived Zenodo JSON project documents.

## When NOT to use

- The metadata value you need to filter on is free text entered in an 'Other' box — such text is inherently not machine-readable, so substring verification cannot be relied upon
- You need ontology-based, reliable selection by specific metadata (e.g. only metagenomic datasets from aquatic invertebrates, or only a specific culture medium) rather than free-text keyword matching
- The target records are BGC–MS/MS links (MIBiG identifier + MS/MS URL + scan number) rather than project-level extraction/sample-preparation metadata

## Inputs

- Archived Zenodo JSON documents of PoDP projects (with modular extraction-method and sample-preparation metadata fields)
- Keyword query string 'methanol + cells' executed via the platform's List-tab search

## Outputs

- methanol_cells_projects.json — verified matching projects with their extraction/sample-preparation metadata, in the platform's project JSON document structure

## How to apply

Load the archived Zenodo JSON project documents, which contain modular extraction-method and sample-preparation metadata fields (the PoDP archives community-curated data to Zenodo in JSON format). Execute the platform's free-text search semantics — the List-tab keyword index backed by Elasticsearch (v7.6.2) — with the query 'methanol + cells' to obtain the candidate project set. For each returned project, inspect its extraction-method and sample-preparation fields and retain only those where both query terms are present as substrings, excluding search hits whose fields do not contain the terms. Write the verified matches to methanol_cells_projects.json in the platform's project document structure, so the search result can be re-checked offline.

## Related tools

- **Paired Omics Data Platform** (Provides the List-tab keyword search ('methanol + cells') over project extraction/sample-preparation text and the project JSON documents whose metadata fields are verified) — https://github.com/iomega/paired-data-form
- **Zenodo** (Source of the regularly archived, downloadable JSON project dataset used for offline field verification)

## Examples

```
python -c "import json;d=json.load(open('projects.json'));m=[p for p in d if 'methanol' in json.dumps(p['extraction_method'].lower()) and 'cells' in json.dumps(p['sample_preparation'].lower())];json.dump(m,open('methanol_cells_projects.json','w'))"
```

## Evaluation signals

- Every retained project's extraction/sample-preparation metadata fields contain both query terms as substrings ('methanol' AND 'cells')
- Projects returned by the search whose fields do not contain the query terms are excluded from the output
- Output JSON conforms to the platform's project document structure (schema at app/public/schema.json in iomega/paired-data-form)
- Verified project set reproduces the article's claim that 'methanol + cells' retrieves projects that used methanol to extract cell pellets
- No false positives: records matching the terms in unrelated fields (e.g. instrumentation method) but not in extraction/sample-preparation fields are not retained

## Limitations

- Free text entered in the 'Other' boxes is inherently not machine-readable, so extraction-method details recorded there cannot be reliably verified by substring checks
- Keyword search operates over the full project document (Elasticsearch full-text index), so a search hit does not by itself guarantee the terms occur in the extraction-method or sample-preparation fields — per-field verification is required
- Capturing the full range of relevant experimental variables in a standardized, machine-readable format was deliberately avoided in the PoDP schema to keep data entry manageable, limiting the precision of metadata-based filtering

## Evidence

- [abstract] Keyword search retrieves methanol-extraction projects: "searching “methanol + cells” retrieves projects that used methanol to extract cell pellets"
- [abstract] List-tab keyword search entry point: "users can search for projects under the “List” tab, using keywords to find studies of interest"
- [abstract] Zenodo JSON archive of projects: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [readme] Elasticsearch-backed full text search: [evidence span withheld — anchor status: external_doc]
- [abstract] Free-text Other fields not machine-readable: "Free text entered in the “Other” boxes is inherently not machine-readable but gives an option for customization by the user"
