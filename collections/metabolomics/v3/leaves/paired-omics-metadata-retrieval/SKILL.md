---
name: paired-omics-metadata-retrieval
description: 'Use when an analysis needs systematically curated BGC–metabolite linkages:
  e.g. assembling validated paired datasets for NPLinker scoring, benchmarking link-ranking
  algorithms, or finding projects by genus, extraction method, or metadata ontology
  terms.'
when_to_use_negative:
- You need raw MS/MS spectra or genome sequence data themselves — the PoDP stores
  no metabolomics or genomics datasets, only metadata linking records in existing
  public databases.
- You need machine-readable values for fields entered as free text in 'Other' boxes
  — these are inherently not machine-readable.
- You expect high-throughput automated pairing of new datasets — the manual burden
  of paired data creation is likely too significant for high-throughput needs.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Paired Omics Data Platform
  role: Source of community-curated metadata linking (meta)genomes with MS/MS spectra;
    projects searchable via List tab and downloadable as JSON
  repo: https://github.com/iomega/paired-data-form
- name: NPLinker
  role: Downstream framework that consumes the assembled paired datasets and computes
    the newly proposed BGC–metabolite scores
- name: Zenodo
  role: Regular archive of all PoDP project JSON records, deposited monthly by the
    platform's redis-queue jobs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:3dd9c0d8d911f17d6ef24e8d5df12e0d086d262dcf99b52afa24fac020c1e160
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - NPLinker
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# paired-omics-metadata-retrieval

## Summary

Retrieve community-curated paired genomic–metabolomic project metadata from the Paired Omics Data Platform (PoDP), via the per-project download button or the regularly archived Zenodo JSON dumps, and resolve the linked public records (GNPS-MassIVE/MetaboLights MS/MS accessions, NCBI GenBank or JGI Genome IDs, MIBiG identifiers) so downstream tools such as NPLinker can score BGC–metabolite links.

## When to use

Use when an analysis needs systematically curated BGC–metabolite linkages: e.g. assembling validated paired datasets for NPLinker scoring, benchmarking link-ranking algorithms, or finding projects by genus, extraction method, or metadata ontology terms.

## When NOT to use

- You need raw MS/MS spectra or genome sequence data themselves — the PoDP stores no metabolomics or genomics datasets, only metadata linking records in existing public databases.
- You need machine-readable values for fields entered as free text in 'Other' boxes — these are inherently not machine-readable.
- You expect high-throughput automated pairing of new datasets — the manual burden of paired data creation is likely too significant for high-throughput needs.

## Inputs

- PoDP project metadata in JSON format (Zenodo archive or per-project download from pairedomicsdata.bioinformatics.nl)
- JSON schema describing the project format (app/public/schema.json in iomega/paired-data-form)
- PoDP web search keywords / ontology term filters (genera, extraction methods, ionization mode)
- PoDP OpenAPI v3.0.3 service specification for programmatic submission/retrieval

## Outputs

- Parsed set of validated BGC–metabolite links (MIBiG identifier + MS/MS URL + scan number + genome label)
- Resolved public record accessions: GNPS-MassIVE / MetaboLights study IDs, NCBI GenBank accessions / JGI Genome IDs, BioSample accession IDs
- Ranked BGC–metabolite score table (bgc_metabolite_scores.tsv) once loaded into NPLinker

## How to apply

Query PoDP (pairedomicsdata.bioinformatics.nl) under the 'List' tab with keywords — e.g. 'Streptomyces | Salinispora' to find the ~18 projects measuring those genera, or 'methanol + cells' for methanol cell-pellet extractions — or use ontology-based filters to reliably select e.g. only metagenomic datasets from aquatic invertebrates or only positive ionization mode. Download each project's metadata as JSON from the per-project download button or from the regularly refreshed Zenodo archive (Zenodo deposition compiled by the iomega/paired-data-form service, which schedules monthly uploads). Parse the JSON to extract validated BGC–metabolite links: MIBiG identifiers paired with MS/MS URLs and scan numbers, user-specified genome labels, and BioSample accession IDs that bridge additional metadata. Resolve each referenced public record — GNPS-MassIVE or MetaboLights study identifiers for metabolomics, NCBI GenBank accessions or JGI Genome IDs for genomes (the platform's redis-queue jobs already fetch species names from GenBank, and an OpenAPI v3.0.3 specification supports programmatic retrieval). Load the assembled paired datasets into NPLinker and rank candidate BGC–metabolite pairs by the newly proposed scores; the PoDP itself stores no omics data, only the linking metadata, so record resolution is mandatory.

## Related tools

- **Paired Omics Data Platform** (Source of community-curated metadata linking (meta)genomes with MS/MS spectra; projects searchable via List tab and downloadable as JSON) — https://github.com/iomega/paired-data-form
- **NPLinker** (Downstream framework that consumes the assembled paired datasets and computes the newly proposed BGC–metabolite scores)
- **Zenodo** (Regular archive of all PoDP project JSON records, deposited monthly by the platform's redis-queue jobs)

## Examples

```
curl -O https://zenodo.org/record/6091663/files/paired_data.json && python -c "import json; projects=json.load(open('paired_data.json')); print([(p['genome_label'], p['mibig_id'], p['ms2_url']) for p in projects])"
```

## Evaluation signals

- Every downloaded project JSON validates against app/public/schema.json from the paired-data-form repository.
- Each extracted BGC–MS/MS link contains a MIBiG identifier, an MS/MS URL, and a scan number.
- Every validated BGC–metabolite link from the demonstration datasets receives a score in bgc_metabolite_scores.tsv when loaded into NPLinker.
- Resolved genome records map to a public database identifier (NCBI GenBank accession or JGI Genome ID), and MS/MS records map to a MassIVE accession or MetaboLights study identifier.
- Retrieval counts match published PoDP scale as a sanity bound: ~70 paired projects, >4,800 genome–metabolite links, and 114 BGC–MS/MS links at time of publication.

## Limitations

- Free text entered in 'Other' boxes is inherently not machine-readable but gives an option for customization by the user.
- Capturing the full range of relevant variables in a standardized, machine-readable format would lead to a very complex and tedious data entry process, so metadata coverage is intentionally minimal and modular.
- The PoDP does not store any metabolomics or genomics datasets itself — it only captures metadata defining pairs of omics datasets in existing public databases.

## Evidence

- [abstract] The PoDP does not store any metabolomics or genomics datasets, but captures metadata defining pairs of omics datasets in existing public databases: "The PoDP does not store any metabolomics or genomics datasets, but captures metadata defining pairs of omics datasets in existing public databases"
- [abstract] Zenodo JSON archive: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [abstract] BGC–MS/MS link creation: "To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number"
- [abstract] Genus keyword search: "searching for the genera (“Streptomyces | Salinispora”) will result in the projects (currently 18) that measured Streptomyces or Salinispora strains"
- [readme] README: architecture and identifier resolution: [evidence span withheld — anchor status: external_doc]
- [readme] README: OpenAPI programmatic access: [evidence span withheld — anchor status: not_found]
