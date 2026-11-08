---
name: json-project-record-parsing
description: Use when a task needs the PoDP's community-curated project records in
  machine-readable form, e.g. to reproduce List-tab keyword searches ('Streptomyces
  | Salinispora', 'methanol + cells') or audit extraction/genome metadata across the
  70 paired datasets.
when_to_use_negative:
- The target information lives in free text entered in the 'Other' boxes of project
  records, which is inherently not machine-readable and cannot be reliably parsed
- The goal is to access raw metabolomics or genomics data itself — the PoDP captures
  only metadata defining pairs of omics datasets held in external public databases
  (MassIVE, MetaboLights, NCBI GenBank/JGI)
- The task requires the current live project set rather than a publication-time snapshot,
  since the Zenodo archive is a periodically frozen dump
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Paired Omics Data Platform
  role: Source of the project JSON records and the schema/full-text search semantics
    that offline parsing must mirror
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Hosts the regularly archived JSON download of all community-curated project
    records used as the parsing input
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:f0b363c8be4229ab61fbb9702f25ea5cd50a46921dac6bbdced6e4b9b9a26842
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# json-project-record-parsing

## Summary

Parses the community-curated Paired Omics Data Platform project records from the regularly archived Zenodo JSON dataset, making modular project metadata (genome labels, MassIVE/MetaboLights accessions, sample preparation, extraction and instrumentation methods) machine-readable for offline filtering and keyword-search reproduction.

## When to use

Use when a task needs the PoDP's community-curated project records in machine-readable form, e.g. to reproduce List-tab keyword searches ('Streptomyces | Salinispora', 'methanol + cells') or audit extraction/genome metadata across the 70 paired datasets.

## When NOT to use

- The target information lives in free text entered in the 'Other' boxes of project records, which is inherently not machine-readable and cannot be reliably parsed
- The goal is to access raw metabolomics or genomics data itself — the PoDP captures only metadata defining pairs of omics datasets held in external public databases (MassIVE, MetaboLights, NCBI GenBank/JGI)
- The task requires the current live project set rather than a publication-time snapshot, since the Zenodo archive is a periodically frozen dump

## Inputs

- Zenodo JSON archive of Paired Omics Data Platform project records (e.g. Zenodo deposition 10.5281/zenodo.6091663)
- Project JSON schema (app/public/schema.json) used to validate record structure

## Outputs

- Parsed list/dict of PoDP project records with genome labels, MassIVE/MetaboLights accessions, BioSample IDs and modular method metadata
- Filtered subsets of matching project identifiers (e.g. methanol_cells_projects.json written in the platform's project document structure)

## How to apply

Download the archived Zenodo JSON dataset (Zenodo deposition maintained by monthly scheduled uploads from the paired-data-form web service) containing one JSON document per PoDP project. Load each record and validate it against the platform's JSON schema (app/public/schema.json in the iomega/paired-data-form repository), which describes the format of a project and its modular metadata fields. Identify the fields relevant to the downstream filter — genome labels and organism/genus text for genus queries, and sample preparation/extraction method fields for method queries such as 'methanol + cells'. When emulating the platform's List-tab search, apply free-text/keyword semantics over these fields (the live platform backs this with an Elasticsearch v7.6.2 full-text index), collecting project identifiers whose records match the query terms. Emit the matched subset (e.g. methanol_cells_projects.json) preserving the platform's project document structure so results can be diffed against live platform output.

## Related tools

- **Paired Omics Data Platform** (Source of the project JSON records and the schema/full-text search semantics that offline parsing must mirror) — https://github.com/iomega/paired-data-form
- **Zenodo** (Hosts the regularly archived JSON download of all community-curated project records used as the parsing input)

## Examples

```
import json; projects = json.load(open('podp_projects.json')); methanol_cells = [p for p in projects if 'methanol' in json.dumps(p).lower() and 'cells' in json.dumps(p).lower()]
```

## Evaluation signals

- Every parsed record validates against the platform's project JSON schema (app/public/schema.json)
- Number of parsed records matches the number of community-curated projects in the archive (70 paired datasets at publication time)
- Reproduced genus filter 'Streptomyces | Salinispora' yields exactly 18 matching projects (the reported count at publication time)
- For the 'methanol + cells' filter, every retained project's extraction/sample-preparation fields contain the substrings 'methanol' and 'cells', and non-matching records are excluded
- Output JSON retains the platform's project document structure so it can be compared record-by-record against live List-tab search results

## Limitations

- Free text entered in the 'Other' boxes of project records is inherently not machine-readable, so parsing cannot capture user-customized metadata
- Offline keyword filtering approximates but does not exactly replicate the live platform's Elasticsearch full-text index semantics
- Match counts are publication-time snapshots (e.g. 18 projects for 'Streptomyces | Salinispora') and will drift as new projects are curated and archived
- The archive contains metadata only; underlying MS/MS and genome data must be fetched separately from MassIVE, MetaboLights or GenBank/JGI identifiers

## Evidence

- [abstract] Zenodo archive availability: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [abstract] Project structure: "The metadata is organized in projects that can consist of multiple related experiments, identified by their MassIVE accession or MetaboLights study identifier"
- [readme] Schema location: "The [JSON schema (app/public/schema.json)](app/public/schema.json) describes the format of an project."
- [readme] Storage model: [evidence span withheld — anchor status: external_doc]
- [readme] Search backend: [evidence span withheld — anchor status: external_doc]
