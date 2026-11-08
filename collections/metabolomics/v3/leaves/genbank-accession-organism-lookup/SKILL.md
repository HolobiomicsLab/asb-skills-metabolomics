---
name: genbank-accession-organism-lookup
description: Use when you have PoDP project JSON entries pairing user-created genome
  labels with NCBI GenBank accession numbers or JGI Genome IDs and need the scientific
  species/organism names for downstream linking, filtering, or analysis.
when_to_use_negative:
- The dataset records contain only free-text 'Other' metadata fields with no machine-readable
  public database identifiers (NCBI GenBank accession or JGI Genome ID) to resolve
- You need metabolomics-side metadata (MassIVE or MetaboLights accession resolution)
  rather than organism names for genomic sources
- The genome identifiers are already enriched with organism names in the PoDP project
  JSON and only pass-through is needed
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: Paired Omics Data Platform
  role: 'Source of the project records: genome entries pair user-created genome labels
    with public database identifiers; projects are archived monthly to Zenodo as JSON'
  repo: https://github.com/iomega/paired-data-form
- name: NCBI GenBank
  role: 'Lookup target for NCBI GenBank accession numbers: E-utilities (esummary/efetch,
    nucleotide database) return the official organism name'
- name: JGI Genome ID
  role: Alternative identifier type resolved via the JGI Genome Portal / IMG API to
    retrieve organism names when the project record uses a JGI identifier instead
    of a GenBank accession
- name: Zenodo
  role: Hosts the regularly archived JSON dump of community-curated PoDP project data
    that serves as the input to the enrichment step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:09001bebe34d9d568af9a3da49243249707bfe627e90a6caf6a9b8bde6cfc3c6
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
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# genbank-accession-organism-lookup

## Summary

Enriches Paired Omics Data Platform (PoDP) project records with official organism names by resolving each (meta)genome's public database identifier — NCBI GenBank accession or JGI Genome ID — against NCBI E-utilities or the JGI/IMG APIs, producing a keyed enrichment table (genome_organism_enrichment.csv).

## When to use

Use when you have PoDP project JSON entries pairing user-created genome labels with NCBI GenBank accession numbers or JGI Genome IDs and need the scientific species/organism names for downstream linking, filtering, or analysis.

## When NOT to use

- The dataset records contain only free-text 'Other' metadata fields with no machine-readable public database identifiers (NCBI GenBank accession or JGI Genome ID) to resolve
- You need metabolomics-side metadata (MassIVE or MetaboLights accession resolution) rather than organism names for genomic sources
- The genome identifiers are already enriched with organism names in the PoDP project JSON and only pass-through is needed

## Inputs

- PoDP project JSON documents (from the Zenodo-archived JSON dump of the Paired Omics Data Platform)
- Genome entries containing user-created genome labels paired with public database identifiers (NCBI GenBank accession numbers or JGI Genome IDs)

## Outputs

- genome_organism_enrichment.csv keyed by project identifier and genome label, with columns for database type, accession, organism name, and fetch status

## How to apply

Download the archived Zenodo JSON dump of PoDP projects (the platform uploads projects to Zenodo each month via a scheduled redis-queue job) and parse each project document to extract every (meta)genome entry, capturing the user-specified genome label plus its public database identifier. For NCBI GenBank accessions, batch-query NCBI E-utilities (esummary/efetch against the nucleotide database) keeping requests at or below 3 per second to retrieve the official organism name; the platform itself performs this same lookup, fetching 'the scientific species name ... from GenBank using the public genome identifiers in the project.' For JGI Genome IDs, query the JGI Genome Portal / IMG API instead, and explicitly record any identifier that cannot be resolved. Assemble results into a table keyed by project identifier and genome label with columns for database type, accession, organism name, and fetch status; deduplicate repeated identifiers before writing genome_organism_enrichment.csv.

## Related tools

- **Paired Omics Data Platform** (Source of the project records: genome entries pair user-created genome labels with public database identifiers; projects are archived monthly to Zenodo as JSON) — https://github.com/iomega/paired-data-form
- **NCBI GenBank** (Lookup target for NCBI GenBank accession numbers: E-utilities (esummary/efetch, nucleotide database) return the official organism name)
- **JGI Genome ID** (Alternative identifier type resolved via the JGI Genome Portal / IMG API to retrieve organism names when the project record uses a JGI identifier instead of a GenBank accession)
- **Zenodo** (Hosts the regularly archived JSON dump of community-curated PoDP project data that serves as the input to the enrichment step)

## Examples

```
curl 'https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?db=nucleotide&id=GCA_000123455.1&retmode=json'  # ≤3 req/sec per accession batch
```

## Evaluation signals

- Every resolved row in genome_organism_enrichment.csv carries a valid (project identifier, genome label) key matching an entry in the source PoDP JSON
- Each NCBI accession resolves to an organism name via the nucleotide database without exceeding 3 E-utilities requests per second
- Rows for JGI Genome IDs have database_type='JGI' and either an organism name or an explicit fetch-status failure entry — no silently missing IDs
- Repeated identifiers are deduplicated so each unique accession appears once with consistent organism names across projects
- Row counts reconcile: number of unique genome entries in the parsed PoDP projects equals number of rows in the enrichment table (resolved + failed)

## Limitations

- JGI Genome IDs that cannot be resolved by the JGI/IMG API must be recorded as explicit failures; not every public identifier in a project yields an organism name
- Free-text metadata in PoDP 'Other' boxes is 'inherently not machine-readable', so genome labels alone may be ambiguous without the paired public identifier
- The enrichment reflects the PoDP snapshot at time of retrieval; the platform re-archives to Zenodo monthly, so newer projects may be absent from an older dump

## Evidence

- [readme] For example, the scientific species name is fetched from GenBank using the public genome identifiers in the project.: [evidence span withheld — anchor status: external_doc]
- [abstract] Genome identifiers and Zenodo archiving: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [abstract] Identifier types supported: "a public database identifier (e.g., a NCBI GenBank accession number or JGI Genome ID)"
