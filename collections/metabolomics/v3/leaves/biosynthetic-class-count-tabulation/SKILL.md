---
name: biosynthetic-class-count-tabulation
description: Use when reproducing or summarizing repository-level counts from MIBiG's
  statistics page — e.g. totals per biosynthetic class or per taxon from the 2021
  curated BGCs in MIBiG 2.0.
when_to_use_negative:
- Counting candidate/predicted BGCs from genome mining output (e.g. antiSMASH results
  on new genomes) — MIBiG only covers clusters of known function
- The goal is per-cluster annotation detail (accession, structure, references) rather
  than repository-level aggregate counts
- The target repository version differs from MIBiG 2.0 (1170 entries pre-update, 851
  added over five years) — counts will not match
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3364
tools:
- name: MIBiG Repository
  role: Web repository whose statistics page and annotation-metadata filters provide
    the entry, class, and taxon counts that are tabulated
  repo: https://mibig.secondarymetabolites.org/
- name: mibig-api
  role: REST-like web API backing the repository database that can serve the underlying
    MIBiG data accessed during tabulation
  repo: https://github.com/mibig-secmet/mibig-api/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:c780ee81274e37f877dcbf252c98779b8a28ed981817e06e361307e5fd8b9034
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIBiG Repository
  - mibig-api
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3364
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# biosynthetic-class-count-tabulation

## Summary

Tabulates curated entry counts from the MIBiG repository statistics page into a structured summary table covering total BGCs (2021 in version 2.0), the seven structure-based biosynthetic classes (polyketide, NRP, hybrids, 'Other'), and taxonomic distributions.

## When to use

Use when reproducing or summarizing repository-level counts from MIBiG's statistics page — e.g. totals per biosynthetic class or per taxon from the 2021 curated BGCs in MIBiG 2.0.

## When NOT to use

- Counting candidate/predicted BGCs from genome mining output (e.g. antiSMASH results on new genomes) — MIBiG only covers clusters of known function
- The goal is per-cluster annotation detail (accession, structure, references) rather than repository-level aggregate counts
- The target repository version differs from MIBiG 2.0 (1170 entries pre-update, 851 added over five years) — counts will not match

## Inputs

- MIBiG repository statistics page (https://mibig.secondarymetabolites.org/) counts for BGC entries, biosynthetic classes, and taxa
- Repository page filter/query results (rows filtered by species name or biosynthetic class)
- MIBiG 2.0 article (DOI 10.1093/nar/gkz882) reported figures for cross-checking

## Outputs

- Structured summary table (CSV, e.g. mibig2_summary_stats.csv) with columns: statistic category (total/class/taxon), label, count
- Per-class counts: Polyketide (825), NRP (627), hybrid classes, 'Other'
- Per-taxon counts: Streptomyces (568), Aspergillus (79), Pseudomonas (61), plants (19)

## How to apply

Access the MIBiG Repository at https://mibig.secondarymetabolites.org/ and locate its statistics page, which exposes published counts of BGC entries, biosynthetic class distributions, and taxonomic distributions. Extract the total curated BGC count (2021), per-class counts distinguishing polyketide (825), NRP (627), hybrid classes and 'Other', and per-taxon counts such as Streptomyces (568), Aspergillus (79), Pseudomonas (61), and plants (19). The repository supports filtering rows by annotation metadata — species names or biosynthetic classes — so counts for subqueries (e.g. complete RiPP BGCs from Streptomyces) can be reproduced with its filter/Boolean query features. Tabulate each value into a structured table with columns for statistic category (total/class/taxon), label, and count. Cross-check extracted values against the figures reported in the MIBiG 2.0 article, then export the table (e.g. mibig2_summary_stats.csv).

## Related tools

- **MIBiG Repository** (Web repository whose statistics page and annotation-metadata filters provide the entry, class, and taxon counts that are tabulated) — https://mibig.secondarymetabolites.org/
- **mibig-api** (REST-like web API backing the repository database that can serve the underlying MIBiG data accessed during tabulation) — https://github.com/mibig-secmet/mibig-api/

## Evaluation signals

- Total curated BGC count equals 2021 for MIBiG 2.0 (1170 original + 851 added over five years)
- Class counts sum to 2021 with Polyketide = 825 and NRP = 627 as the two largest classes
- Taxon counts include Streptomyces = 568, Aspergillus = 79, Pseudomonas = 61, and exactly 19 plant-derived BGCs
- Extracted values match the figures reported in the MIBiG 2.0 article text
- Output CSV has a consistent schema: statistic category (total/class/taxon), label, count — with no duplicate rows

## Limitations

- Counts are version-specific to MIBiG 2.0 and will drift as the repository is updated; 851 entries were added over five years alone
- Repository filters operate on annotation metadata (species names, biosynthetic classes), so counts depend on annotation quality; MIBiG 2.0 itself required fixing empty or mistyped information and removing duplicate entries
- Only curated BGCs of known function are counted — clusters in databases like IMG-ABC or antiSMASH-DB (many thousands of candidates) are out of scope

## Evidence

- [abstract] we improved the user experience by adding new features such as query searches and a statistics page, and: "we improved the user experience by adding new features such as query searches and a statistics page, and"
- [intro] Rows in the 'Repository' page can now be filtered and sorted based on annotation metadata: "Rows in the ‘Repository’ page can now be filtered and sorted based on annotation metadata"
- [supplementary] The MIBiG Repository is available at https://mibig.secondarymetabolites.org/.: "The MIBiG Repository is available at https://mibig.secondarymetabolites.org/."
- [readme] The MIBiG API server is licensed under the Apache License version 2: "The MIBiG API server is licensed under the Apache License version 2"
