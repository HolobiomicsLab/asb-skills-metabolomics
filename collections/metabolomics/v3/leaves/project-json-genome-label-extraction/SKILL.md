---
name: project-json-genome-label-extraction
description: Use when PoDP project JSON (e.g., from the monthly Zenodo archive) must
  be parsed to enumerate genome entries by genome label and resolve their organism
  names from public genome identifiers.
when_to_use_negative:
- The goal is to find projects by keyword or metadata filters (e.g., 'Streptomyces
  | Salinispora' genera or 'methanol + cells' extraction method) via the PoDP List
  tab rather than to enumerate genome labels per project
- The input is already an enrichment table and only needs joining or deduplication,
  not identifier-to-organism resolution
- The organism names are needed from sources other than the genome identifiers (e.g.,
  BioSample metadata linking), which this skill does not resolve
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Paired Omics Data Platform
  role: Source of the project JSON records containing genome label + public identifier
    pairs; also offers an OpenAPI v3.0.3 web service for programmatic submission and
    retrieval of projects
  repo: https://github.com/iomega/paired-data-form
- name: NCBI GenBank
  role: Reference database queried via E-utilities to resolve the official organism
    name for NCBI GenBank accession numbers
- name: JGI Genome ID
  role: Alternative public genome identifier system whose IDs are resolved to organism
    names via the JGI Genome Portal/IMG API
- name: Zenodo
  role: Host of the regularly archived, downloadable JSON dump of all community-curated
    PoDP project data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:3289a157aa239938edeb99961f60eea2e6c29ac63c4169d6fb443c035d31c6e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - JGI Genome ID
  - NCBI GenBank
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# project-json-genome-label-extraction

## Summary

Extracts every (meta)genome entry from Paired Omics Data Platform project JSON documents — pairing the user-created genome label with its public database identifier (NCBI GenBank accession or JGI Genome ID) — and enriches them with official organism names fetched from GenBank/JGI.

## When to use

Use when PoDP project JSON (e.g., from the monthly Zenodo archive) must be parsed to enumerate genome entries by genome label and resolve their organism names from public genome identifiers.

## When NOT to use

- The goal is to find projects by keyword or metadata filters (e.g., 'Streptomyces | Salinispora' genera or 'methanol + cells' extraction method) via the PoDP List tab rather than to enumerate genome labels per project
- The input is already an enrichment table and only needs joining or deduplication, not identifier-to-organism resolution
- The organism names are needed from sources other than the genome identifiers (e.g., BioSample metadata linking), which this skill does not resolve

## Inputs

- Zenodo JSON dump of Paired Omics Data Platform project records
- Individual PoDP project JSON documents following the paired-data-form schema (schema.json)
- Genome entries containing NCBI GenBank accession numbers or JGI Genome IDs paired with user-created genome labels

## Outputs

- genome_organism_enrichment.csv keyed by project identifier and genome label
- Table columns: database type, accession, organism name, fetch status
- Explicit failure records for identifiers that could not be resolved

## How to apply

Download the regularly archived Zenodo JSON dump of PoDP projects (uploaded monthly by the platform's redis-queued jobs) and enumerate the individual project JSON documents. Parse each project and extract each (meta)genome entry, capturing the user-specified genome label together with its public database identifier — an NCBI GenBank accession number or a JGI Genome ID. For GenBank accessions, query NCBI E-utilities (esummary/efetch against the nucleotide database, throttled to ≤3 requests/second) to retrieve the scientific species name; for JGI Genome IDs, query the JGI Genome Portal/IMG API, explicitly recording unresolved IDs. Deduplicate repeated identifiers and write the result as a table keyed by project identifier and genome label, with columns for database type, accession, organism name, and fetch status.

## Related tools

- **Paired Omics Data Platform** (Source of the project JSON records containing genome label + public identifier pairs; also offers an OpenAPI v3.0.3 web service for programmatic submission and retrieval of projects) — https://github.com/iomega/paired-data-form
- **NCBI GenBank** (Reference database queried via E-utilities to resolve the official organism name for NCBI GenBank accession numbers)
- **JGI Genome ID** (Alternative public genome identifier system whose IDs are resolved to organism names via the JGI Genome Portal/IMG API)
- **Zenodo** (Host of the regularly archived, downloadable JSON dump of all community-curated PoDP project data)

## Examples

```
from Bio import Entrez; Entrez.email='<maintainer e-mail, see the repository README>'; h=Entrez.esummary(db='nucleotide', id=','.join(accessions)); print(Entrez.read(h))
```

## Evaluation signals

- Every genome entry in each parsed project JSON appears exactly once in the output, keyed by (project identifier, genome label) after deduplication of repeated identifiers
- Each output row records the database type correctly as GenBank or JGI, matching the identifier format (e.g., a GenBank accession number vs a JGI Genome ID)
- Rows where the organism name could not be fetched carry an explicit non-empty fetch status rather than a blank name
- NCBI E-utilities request rate stays at or below 3 requests per second throughout the enrichment run
- Spot-check a sample of resolved GenBank accessions: the fetched scientific species name matches the organism shown on the GenBank record

## Limitations

- Free text entered in 'Other' boxes in PoDP project records is inherently not machine-readable, so some genome-related metadata may not parse cleanly
- The PoDP stores no metabolomics or genomics datasets itself, only metadata — organism names must be fetched live from GenBank/JGI, and unresolvable identifiers must be handled explicitly
- Zenodo archiving of the curated data happens on a monthly schedule, so the JSON dump may lag the most recent community-curated submissions

## Evidence

- [readme] For example, the scientific species name is fetched from GenBank using the public genome identifiers in the project.: [evidence span withheld — anchor status: external_doc]
- [abstract] Zenodo JSON archive of curated data: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [abstract] Genome identifier types: "a public database identifier (e.g., a NCBI GenBank accession number or JGI Genome ID)"
- [readme] Scheduled enrichment/upload jobs: [evidence span withheld — anchor status: external_doc]
