---
name: public-data-deposit-retrieval
description: Use when a question asks how many PoDP projects match a List-tab keyword
  query (e.g., 'Streptomyces | Salinispora' → 18 projects) and you must retrieve the
  archived Zenodo JSON snapshot and filter it locally.
when_to_use_negative:
- You need the actual raw metabolomics or genomics data files — the PoDP does not
  store datasets, it only captures metadata linking to public repositories (MassIVE,
  MetaboLights, NCBI GenBank)
- You need reproducible ontology-based metadata selection across arbitrary fields
  with guaranteed machine-readable semantics — free text entered in 'Other' boxes
  is inherently not machine-readable
- You expect the current live count to equal a historical figure — keyword counts
  (e.g., 'currently 18') change as the Zenodo archive is re-uploaded monthly
edam_operation: http://edamontology.org/operation_2422
tools:
- name: Paired Omics Data Platform
  role: Source of the community-curated project records; its List-tab keyword search
    semantics are mirrored in the offline filter
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Hosts the regularly archived JSON snapshot of all PoDP project records that
    is downloaded for offline filtering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:68a495ed52254f44b582a8eda6ff6bc11c0005728ac686653e148fcc29fb5b11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# public-data-deposit-retrieval

## Summary

Retrieves the community-curated Paired Omics Data Platform (PoDP) project archive from its regularly published Zenodo dataset (JSON format) and applies the platform's List-tab keyword search semantics (e.g., 'Streptomyces | Salinispora') offline to reproduce project counts and retrieve matching project records.

## When to use

Use when a question asks how many PoDP projects match a List-tab keyword query (e.g., 'Streptomyces | Salinispora' → 18 projects) and you must retrieve the archived Zenodo JSON snapshot and filter it locally.

## When NOT to use

- You need the actual raw metabolomics or genomics data files — the PoDP does not store datasets, it only captures metadata linking to public repositories (MassIVE, MetaboLights, NCBI GenBank)
- You need reproducible ontology-based metadata selection across arbitrary fields with guaranteed machine-readable semantics — free text entered in 'Other' boxes is inherently not machine-readable
- You expect the current live count to equal a historical figure — keyword counts (e.g., 'currently 18') change as the Zenodo archive is re-uploaded monthly

## Inputs

- Archived Zenodo dataset (DOI 10.5281/zenodo.6091663 series) of PoDP project records in JSON format
- Keyword query string using the List-tab search syntax (e.g., 'Streptomyces | Salinispora', 'methanol + cells')

## Outputs

- Subset of matching project identifiers/JSON project records
- Count of matched projects comparable to the published reference value (18 for 'Streptomyces | Salinispora' at publication time)

## How to apply

Download the archived Zenodo dataset of the Paired Omics Data Platform, which contains all community-curated project records as JSON files validated against the schema at app/public/schema.json in the iomega/paired-data-form repository. Each project record is organized as a project with multiple related experiments identified by MassIVE accession or MetaboLights study identifier, with (meta)genomes added via public database identifiers (e.g., NCBI GenBank accession, JGI Genome ID) and user-specified genome labels. Apply a keyword filter across project fields (genome labels, metadata labels, organism/genus text fetched from GenBank) mirroring the List-tab full-text search, which the platform implements via an Elasticsearch (v7.6.2) index — e.g., the OR-style query 'Streptomyces | Salinispora' matches projects that measured strains of either genus. Collect the matching project identifiers, count them, and report the list and count alongside the reference value (18 projects at publication time), noting that the live archive is re-uploaded to Zenodo monthly so counts drift over time.

## Related tools

- **Paired Omics Data Platform** (Source of the community-curated project records; its List-tab keyword search semantics are mirrored in the offline filter) — https://github.com/iomega/paired-data-form
- **Zenodo** (Hosts the regularly archived JSON snapshot of all PoDP project records that is downloaded for offline filtering)

## Examples

```
import json; ps=json.load(open('podp_zenodo.json')); m=[p for p in ps if 'Streptomyces' in json.dumps(p) or 'Salinispora' in json.dumps(p)]; print(len(m))
```

## Evaluation signals

- Keyword filter over the archived JSON returns exactly 18 matching projects for 'Streptomyces | Salinispora' against the publication-time snapshot
- Every matched project record is valid against the JSON schema app/public/schema.json from the iomega/paired-data-form repository
- Matched project identifiers carry genome labels, MassIVE accession or MetaboLights study identifier, and genome public-database identifiers consistent with genus terms
- Running the same filter on a newer Zenodo snapshot yields a count >= the reference (archive grows monotonically via monthly uploads)

## Limitations

- Counts are snapshot-dependent: the reported 18 is 'currently 18' at publication time and the Zenodo archive is refreshed monthly
- Free text entered in the 'Other' boxes is inherently not machine-readable, so keyword matches can miss or over-match such entries
- Offline string matching over JSON only approximates the platform's Elasticsearch (v7.6.2) full-text search semantics

## Evidence

- [abstract] Zenodo archive availability: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [abstract] List-tab keyword search: "users can search for projects under the “List” tab, using keywords to find studies of interest"
- [abstract] Streptomyces | Salinispora reference count: "searching for the genera (“Streptomyces | Salinispora”) will result in the projects (currently 18) that measured Streptomyces or Salinispora strains"
- [abstract] PoDP stores metadata, not datasets: "The PoDP does not store any metabolomics or genomics datasets, but captures metadata defining pairs of omics datasets in existing public databases"
- [readme] Elasticsearch full-text index and monthly Zenodo upload: [evidence span withheld — anchor status: external_doc]
