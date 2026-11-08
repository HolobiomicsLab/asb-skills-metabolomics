---
name: bgc-homology-search-against-mibig-reference
description: Use when you need to benchmark newly mined BGCs (e.g. from uncultivated
  soil bacteria or soil metagenomes) against a curated set of experimentally characterized
  BGCs of known function.
when_to_use_negative:
- Your goal is to mine or store candidate BGCs regardless of known function — IMG-ABC
  or antiSMASH-DB store many thousands of candidate BGCs, whereas MIBiG is restricted
  to BGCs of known function
- Query BGCs are already curated entries in MIBiG, since self-hits trivially inflate
  homology and deflate novelty fractions
- You need experimentally verified product structures with chemical cross-links only
  — MIBiG is a BGC-sequence reference and does not replace spectral/chemical databases
  such as GNPS or the Natural Products Atlas
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MIBiG Repository
  role: Provides the curated reference set of BGCs of known function used as the homology
    baseline
  repo: https://mibig.secondarymetabolites.org/
- name: antiSMASH
  role: Detects the candidate BGCs in genomes/metagenomes that are then searched against
    MIBiG; a customised antiSMASH 5 module sideloads a MIBiG annotation file to generate
    BGC pages
- name: GitHub (mibig-secmet)
  role: Distributes the MIBiG JSON data standard, SQL schema, and source code used
    to build the reference dataset
  repo: https://github.com/mibig-secmet
- name: mibig-api
  role: REST-like web API for programmatic access to the underlying PostgreSQL MIBiG
    database
  repo: https://github.com/mibig-secmet/mibig-api/
- name: PostgreSQL
  role: Relational database backing the MIBiG repository accessed via the API
  repo: https://www.postgresql.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:316b4ccd427d63dc9e71a2dda86da074ed88c56645ec51dc961f0f62fe95365f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - GitHub (mibig-secmet)
  - MIBiG Repository
  - PostgreSQL
  - antiSMASH
  - mibig-api
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# bgc-homology-search-against-mibig-reference

## Summary

Use the MIBiG repository of biosynthetic gene clusters (BGCs) of known function as a homology reference to quantify how novel query BGCs are and to associate metagenomic BGCs with activities such as antibacterial function.

## When to use

Use when you need to benchmark newly mined BGCs (e.g. from uncultivated soil bacteria or soil metagenomes) against a curated set of experimentally characterized BGCs of known function.

## When NOT to use

- Your goal is to mine or store candidate BGCs regardless of known function — IMG-ABC or antiSMASH-DB store many thousands of candidate BGCs, whereas MIBiG is restricted to BGCs of known function
- Query BGCs are already curated entries in MIBiG, since self-hits trivially inflate homology and deflate novelty fractions
- You need experimentally verified product structures with chemical cross-links only — MIBiG is a BGC-sequence reference and does not replace spectral/chemical databases such as GNPS or the Natural Products Atlas

## Inputs

- Query BGC nucleotide/protein sequences, e.g. from antiSMASH runs on genome or metagenome assemblies
- MIBiG 2.0 JSON-formatted BGC entries (biosynthetic class, taxonomic identifier, product annotation fields) from the mibig-secmet GitHub repository or the MIBiG REST-like web API
- Soil metagenome sample metadata, e.g. antimicrobial resistance gene counts for correlation analyses

## Outputs

- Per-query BGC homology calls against MIBiG (match / no homology to any known MIBiG cluster)
- Novelty fraction of query BGCs lacking MIBiG homology
- Identified antibacterial/fungal BGCs and their correlation with antimicrobial resistance gene counts across samples
- Structured CSV collating reproduction metrics (novelty fraction, correlation coefficient)

## How to apply

Download the MIBiG 2.0 data (JSON-formatted entries, expanded to 2021 curated BGCs after the addition of 851 new entries) from the mibig-secmet GitHub repository or via its REST-like web API backed by PostgreSQL, and extract biosynthetic class, taxonomic identifier, and product annotation fields from the JSON. Run homology searches of your query BGCs (e.g. soil-bacterial or fungal BGCs detected with antiSMASH) against the MIBiG reference set to compute the fraction of query clusters lacking homology to any known MIBiG cluster, or to identify MIBiG-matching antibacterial BGCs whose abundance can be correlated with other soil genomic features such as antimicrobial resistance gene counts. Decide on the reference version explicitly: MIBiG 2.0 is the re-annotated, schema-validated release, so homology fractions are only comparable across studies using the same reference build. Collate per-query homology status, best MIBiG match, and downstream metrics (novelty fraction, correlation coefficients) into a structured table, and check that reproduced values fall within the ranges reported in Crits-Cristoph et al. and Bahram et al.

## Related tools

- **MIBiG Repository** (Provides the curated reference set of BGCs of known function used as the homology baseline) — https://mibig.secondarymetabolites.org/
- **antiSMASH** (Detects the candidate BGCs in genomes/metagenomes that are then searched against MIBiG; a customised antiSMASH 5 module sideloads a MIBiG annotation file to generate BGC pages)
- **GitHub (mibig-secmet)** (Distributes the MIBiG JSON data standard, SQL schema, and source code used to build the reference dataset) — https://github.com/mibig-secmet
- **mibig-api** (REST-like web API for programmatic access to the underlying PostgreSQL MIBiG database) — https://github.com/mibig-secmet/mibig-api/
- **PostgreSQL** (Relational database backing the MIBiG repository accessed via the API) — https://www.postgresql.org/

## Examples

```
git clone https://github.com/mibig-secmet/mibig-api && query the MIBiG PostgreSQL reference via the REST API for homology searches of antiSMASH-called BGCs
```

## Evaluation signals

- Reproduced novelty fraction (uncultivated soil-bacterial BGCs lacking MIBiG homology) falls within the range reported by Crits-Cristoph et al.
- Correlation between antibacterial BGC abundance and antimicrobial resistance gene counts falls within the range reported by Bahram et al.
- All parsed MIBiG entries validate against the MIBiG JSON schema (JSON schema description and validation technology)
- Query/reference counts reconcile with the MIBiG 2.0 release size of 2021 curated BGCs (1170 original + 851 added)
- antiSMASH-generated BGC pages correctly sideload the MIBiG annotation file, confirming consistent reference annotations

## Limitations

- Homology-based novelty fractions depend on the MIBiG version used; the 2.0 release added 851 entries and re-annotated the entire database, so fractions are not directly comparable to analyses run against older builds
- MIBiG only contains BGCs of known function; BGCs lacking MIBiG homology may still be homologous to uncharacterized candidate clusters in IMG-ABC or antiSMASH-DB
- Low-quality or duplicate historical entries can bias homology hits — MIBiG 2.0 addressed this via extensive manual curation and removal of duplicates, so older data dumps should not be used

## Evidence

- [intro] Crits-Cristoph et al. used MIBiG as homology baseline for novelty assessment: "Crits-Cristoph et al. (7) recently used MIBiG to assess and highlight the exceptional novelty of BGCs"
- [intro] Bahram et al. used MIBiG homology searches to identify antibacterial fungal BGCs: "Bahram et al. (8) used homology searches against MIBiG to identify fungal BGCs associated with antibacterial activity"
- [intro] MIBiG is the central reference database for BGCs of known function: "the MIBiG repository has become a central reference database for BGCs of known function"
- [intro] API gives programmatic access to the underlying PostgreSQL database: "A REST-like web API (https://github.com/mibig-secmet/mibig-api/) handles access to the underlying PostgreSQL"
- [readme] MIBiG API server exposes the repository programmatically: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene
class (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
