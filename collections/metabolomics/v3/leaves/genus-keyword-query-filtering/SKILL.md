---
name: genus-keyword-query-filtering
description: Use when a research question asks how many PoDP projects match a genus-level
  keyword query (e.g. 'Streptomyces | Salinispora') and the platform's List-tab result
  count must be reproduced from the archived JSON.
when_to_use_negative:
- Selecting datasets by structured/ontology-based metadata (e.g. only metagenomic
  datasets from aquatic invertebrates or only positive ionization mode) — that requires
  the platform's ontology-supported metadata selection, not free-text keyword search
- Searching on values recorded only in free-text 'Other' boxes, which are inherently
  not machine-readable and thus unreliable to match programmatically
- Reproducing searches over the raw metabolomics or genomics datasets themselves —
  the PoDP stores metadata links, not the MS or genome data (those live in MassIVE,
  MetaboLights, NCBI GenBank, JGI)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_0091
tools:
- name: Paired Omics Data Platform
  role: Source of the community-curated paired-omics project records and the List-tab
    keyword search whose semantics the filter mirrors
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Hosts the regularly archived JSON download of all PoDP project records used
    for offline filtering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:5dc66dacd6e588e5aa92b67bc6194cfe2856365da430f3f3ec3b9eb40403f9bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# genus-keyword-query-filtering

## Summary

Reproduces the Paired Omics Data Platform 'List' tab keyword search over community-curated paired omics projects by filtering the regularly archived Zenodo JSON dataset for genus terms (e.g. 'Streptomyces | Salinispora') and counting matching projects.

## When to use

Use when a research question asks how many PoDP projects match a genus-level keyword query (e.g. 'Streptomyces | Salinispora') and the platform's List-tab result count must be reproduced from the archived JSON.

## When NOT to use

- Selecting datasets by structured/ontology-based metadata (e.g. only metagenomic datasets from aquatic invertebrates or only positive ionization mode) — that requires the platform's ontology-supported metadata selection, not free-text keyword search
- Searching on values recorded only in free-text 'Other' boxes, which are inherently not machine-readable and thus unreliable to match programmatically
- Reproducing searches over the raw metabolomics or genomics datasets themselves — the PoDP stores metadata links, not the MS or genome data (those live in MassIVE, MetaboLights, NCBI GenBank, JGI)

## Inputs

- Zenodo-archived PoDP dataset in JSON format (all community-curated project records)
- Pipe-separated keyword query string, e.g. 'Streptomyces | Salinispora'

## Outputs

- List of matching PoDP project identifiers whose records contain either genus term
- Count of matched projects (expected 18 at publication time)

## How to apply

Download the community-curated PoDP project records as JSON from the Zenodo archive, which the platform refreshes monthly via a scheduled redis-queue job. Mirror the List tab's keyword-search semantics by applying the pipe-separated OR query (e.g. 'Streptomyces | Salinispora') across project fields — genome labels, metadata labels, and organism/genus text such as the scientific species names the service fetches from GenBank using each project's public genome identifiers. Collect the matching project identifiers, count them, and compare against the reference count (18 projects measured Streptomyces or Salinispora strains at publication time). Note the query is project-level: a project matches if any of its genome labels or metadata text contains either term, since the platform organizes multiple related experiments (MassIVE or MetaboLights IDs) into a single project record.

## Related tools

- **Paired Omics Data Platform** (Source of the community-curated paired-omics project records and the List-tab keyword search whose semantics the filter mirrors) — https://github.com/iomega/paired-data-form
- **Zenodo** (Hosts the regularly archived JSON download of all PoDP project records used for offline filtering)

## Examples

```
python -c "import json;d=json.load(open('podp_zenodo.json'));m=[p for p in d if 'Streptomyces' in json.dumps(p) or 'Salinispora' in json.dumps(p)];print(len(m))"
```

## Evaluation signals

- The number of matching projects equals the reported value of 18 (count at publication time)
- Every matched project record demonstrably measures a Streptomyces or Salinispora strain (genus term appears in genome label or metadata text)
- The matched project identifiers resolve to project pages on the live PoDP (clickable by project identifier)
- Query semantics match the List tab: pipe-separated terms behave as an OR across fields, unlike '+'-joined queries such as 'methanol + cells'
- The input dataset is the current Zenodo JSON archive (freshness matters — the count is time-sensitive)

## Limitations

- The match count is time-sensitive: 18 is the count at publication time and grows as more laboratories (over 45 labs from 10 countries contributing 70 paired datasets so far) submit projects
- Free text entered in 'Other' boxes is inherently not machine-readable, so keyword filtering may miss projects that recorded genus-relevant information only there
- Scientific species names in project records are fetched from GenBank via public genome identifiers, so search recall depends on that enrichment pipeline having run for each project

## Evidence

- [abstract] Genus query returns 18 projects: "searching for the genera (“Streptomyces | Salinispora”) will result in the projects (currently 18) that measured Streptomyces or Salinispora strains"
- [abstract] List tab keyword search: "users can search for projects under the “List” tab, using keywords to find studies of interest"
- [abstract] Zenodo JSON archive: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [readme] Full-text search implementation and scheduled Zenodo uploads: [evidence span withheld — anchor status: external_doc]
- [readme] Species names fetched from GenBank: [evidence span withheld — anchor status: external_doc]
