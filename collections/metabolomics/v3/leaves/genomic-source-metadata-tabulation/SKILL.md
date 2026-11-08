---
name: genomic-source-metadata-tabulation
description: Use when you have PoDP project JSON records containing (meta)genome entries
  with GenBank accession numbers or JGI Genome IDs and need organism names attached
  per project/genome label for downstream enrichment or filtering.
when_to_use_negative:
- The genome entries contain only free text entered in 'Other' boxes, which is inherently
  not machine-readable and has no resolvable public identifier
- The goal is to link MS/MS spectra URLs or MIBiG identifiers to genomes (BGC–MS/MS
  linking), not to resolve organism names from identifiers
- Input is a metabolomics metadata task (MassIVE/MetaboLights accessions, sample preparation
  or extraction methods) that involves no genomic source identifiers
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3885
tools:
- name: Paired Omics Data Platform
  role: 'Source of project JSON records: each (meta)genome is added to a project via
    a public database identifier with a user-created genome label; the JSON schema
    and OpenAPI spec describe the format'
  repo: https://github.com/iomega/paired-data-form
- name: NCBI GenBank
  role: Reference database whose accessions are resolved to official organism names
    via E-utilities (esummary/efetch, nucleotide database)
- name: JGI Genome ID
  role: Alternative public genome identifier resolved to organism names through the
    JGI Genome Portal/IMG API
- name: Zenodo
  role: Archive hosting the regularly updated JSON dump of community-curated PoDP
    projects used as the enumeration source
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:ea3d56fa445d0cdc109b0703bff0862243320a257617166eb02b48bba0f16d0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - JGI Genome ID
  - NCBI GenBank
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3885
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# genomic-source-metadata-tabulation

## Summary

Reconstructs the PoDP project-enrichment step that resolves public genome identifiers (NCBI GenBank accessions or JGI Genome IDs) to official organism names, tabulated per project and user-created genome label. The PoDP web service performs this same enrichment automatically, but agents can reproduce it offline from the Zenodo JSON archive.

## When to use

Use when you have PoDP project JSON records containing (meta)genome entries with GenBank accession numbers or JGI Genome IDs and need organism names attached per project/genome label for downstream enrichment or filtering.

## When NOT to use

- The genome entries contain only free text entered in 'Other' boxes, which is inherently not machine-readable and has no resolvable public identifier
- The goal is to link MS/MS spectra URLs or MIBiG identifiers to genomes (BGC–MS/MS linking), not to resolve organism names from identifiers
- Input is a metabolomics metadata task (MassIVE/MetaboLights accessions, sample preparation or extraction methods) that involves no genomic source identifiers

## Inputs

- Zenodo JSON archive of Paired Omics Data Platform projects (community-curated dump)
- Individual PoDP project JSON documents containing (meta)genome entries
- Genome labels paired with public database identifiers (NCBI GenBank accession numbers or JGI Genome IDs)

## Outputs

- genome_organism_enrichment.csv keyed by project identifier and genome label, with database type, accession, organism name, and fetch status columns

## How to apply

Download the regularly archived Zenodo JSON dump of PoDP projects and enumerate the individual project documents, each organized under a project with multiple related experiments identified by MassIVE accession or MetaboLights study identifier. Parse each project JSON to extract every (meta)genome entry, capturing the user-specified genome label together with its public database identifier — the same identifier/label pairing the platform records for genome–metabolome links. For GenBank accessions, batch-query NCBI E-utilities (esummary/efetch against the nucleotide database, throttled to ≤3 requests/second) to retrieve the official scientific species name, mirroring what the PoDP web service does via its redis-scheduled enrichment jobs. For JGI Genome IDs, query the JGI Genome Portal/IMG API, explicitly recording failures where an identifier cannot be resolved. Assemble a table keyed by project identifier and genome label with columns for database type, accession, organism name, and fetch status, deduplicating repeated identifiers before writing genome_organism_enrichment.csv.

## Related tools

- **Paired Omics Data Platform** (Source of project JSON records: each (meta)genome is added to a project via a public database identifier with a user-created genome label; the JSON schema and OpenAPI spec describe the format) — https://github.com/iomega/paired-data-form
- **NCBI GenBank** (Reference database whose accessions are resolved to official organism names via E-utilities (esummary/efetch, nucleotide database))
- **JGI Genome ID** (Alternative public genome identifier resolved to organism names through the JGI Genome Portal/IMG API)
- **Zenodo** (Archive hosting the regularly updated JSON dump of community-curated PoDP projects used as the enumeration source)

## Examples

```
python -c "from Bio import Entrez; Entrez.email='<maintainer e-mail, see the repository README>'; r=Entrez.read(Entrez.esummary(db='nucleotide', id='CM000914.1')); print(r[0]['Organism'])"
```

## Evaluation signals

- One row exists per unique (project identifier, genome label) pair after deduplication of repeated public identifiers
- Every row carries an explicit fetch status; JGI Genome IDs that cannot be resolved are flagged rather than silently dropped
- Valid NCBI GenBank accessions yield non-empty organism names matching the NCBI nucleotide record
- E-utilities requests are throttled to ≤3 per second throughout the batch
- Row counts are consistent with the number of (meta)genome entries across enumerated project JSONs (e.g., the archive held 1,306 metagenomes, 1,268 genomes, and 42 MAGs at publication)

## Limitations

- The PoDP enrichment is rate-limited and identifier-dependent: JGI Genome IDs may not resolve, and failures must be recorded explicitly rather than retried indefinitely
- Organism names are fetched at archive time; the platform's redis-scheduled jobs re-fetch monthly and re-upload to Zenodo, so offline tables can go stale relative to the live database
- The skill enriches only organism names from public identifiers — it does not capture sample preparation, extraction, or instrumentation method metadata stored elsewhere via BioSample accession IDs

## Evidence

- [readme] GenBank accession resolution to species name: [evidence span withheld — anchor status: external_doc]
- [readme] Scheduled enrichment and Zenodo upload: [evidence span withheld — anchor status: external_doc]
- [abstract] Public identifier plus genome label: "a public database identifier (e.g., a NCBI GenBank accession number or JGI Genome ID)"
- [abstract] Zenodo JSON archive availability: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [abstract] Genomes added via public identifier within projects: "The (meta)genomes(s) used in these experiments can all be added to the same project via a public database identifier"
