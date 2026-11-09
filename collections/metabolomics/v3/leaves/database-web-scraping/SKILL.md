---
name: database-web-scraping
description: Use when a task requires reproducing published counts or distributions
  (e.g. total curated BGCs, per-class and per-taxon numbers) from a database like
  MIBiG whose web UI exposes a statistics or repository page.
when_to_use_negative:
- A REST-like API exists for the database (e.g. the MIBiG web API) and structured
  programmatic access is preferred over HTML extraction
- The target is candidate/predicted BGCs from genome mining tools like antiSMASH-DB
  or IMG-ABC rather than curated, known-function entries in MIBiG
- The needed statistics are already published in the article text and no per-entry
  filtering or recomputation is required
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MIBiG Repository
  role: Web interface hosting the statistics page and filterable Repository page from
    which counts and distributions are extracted
  repo: https://mibig.secondarymetabolites.org/
- name: MIBiG API
  role: REST-like web API providing programmatic access to the underlying MIBiG PostgreSQL
    database as an alternative to HTML scraping
  repo: https://github.com/mibig-secmet/mibig-api/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:e725bcd3c178a93b8e43c2eb7e18509f4c272604019e6da4f4dc3dfc83d8a73a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIBiG API
  - MIBiG Repository
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# database-web-scraping

## Summary

Extract structured summary statistics (entry counts, class distributions, taxonomic breakdowns) from a scientific database's web interface, such as the MIBiG repository's statistics page, and tabulate them for cross-checking against the companion publication.

## When to use

Use when a task requires reproducing published counts or distributions (e.g. total curated BGCs, per-class and per-taxon numbers) from a database like MIBiG whose web UI exposes a statistics or repository page.

## When NOT to use

- A REST-like API exists for the database (e.g. the MIBiG web API) and structured programmatic access is preferred over HTML extraction
- The target is candidate/predicted BGCs from genome mining tools like antiSMASH-DB or IMG-ABC rather than curated, known-function entries in MIBiG
- The needed statistics are already published in the article text and no per-entry filtering or recomputation is required

## Inputs

- Public database web interface URL (e.g. https://mibig.secondarymetabolites.org/ statistics page)
- Companion publication text for cross-validation of extracted figures

## Outputs

- Structured summary table (e.g. mibig2_summary_stats.csv) with columns: statistic category, label, count
- Per-class and per-taxon count breakdowns for the database release

## How to apply

Navigate to the database's public web interface (for MIBiG, https://mibig.secondarymetabolites.org/) and locate the statistics page, which MIBiG 2.0 added as a new feature alongside query searches. Extract the headline counts — for MIBiG 2.0 this is 2021 manually curated BGCs (expanded from 1170 by 851 new entries) — plus per-class counts (Polyketide 825, NRP 627, hybrid and 'Other' classes) and per-taxon counts (Streptomyces 568, Aspergillus 79, Pseudomonas 61, 19 plant BGCs). Where the UI supports filtering (e.g. the Repository page allows filtering by compound names, taxonomic identifiers or biosynthetic classes, and Boolean queries like 'find all complete RiPP BGCs from the genus Streptomyces'), use those filters to derive sub-counts rather than manual counting. Tabulate extracted values into a structured CSV (columns: statistic category, label, count) and cross-check each figure against the companion article text before saving. For programmatic access, prefer the REST-like web API over HTML scraping when one exists.

## Related tools

- **MIBiG Repository** (Web interface hosting the statistics page and filterable Repository page from which counts and distributions are extracted) — https://mibig.secondarymetabolites.org/
- **MIBiG API** (REST-like web API providing programmatic access to the underlying MIBiG PostgreSQL database as an alternative to HTML scraping) — https://github.com/mibig-secmet/mibig-api/

## Examples

```
curl -s https://mibig.secondarymetabolites.org/api/v1/search -d '{"query":"genus:Streptomyces"}' > mibig_streptomyces.json
```

## Evaluation signals

- Extracted total (2021) equals the sum implied by the article's 'expanded from 1170 to 2021 curated BGCs' (1170 + 851 new entries)
- Per-class counts sum to the total curated BGC count (825 Polyketide + 627 NRP + hybrids + Other = 2021)
- Per-taxon counts (Streptomyces 568, Aspergillus 79, Pseudomonas 61, plants 19) match figures reported in the MIBiG 2.0 article text
- Output CSV parses cleanly with consistent category/label/count columns and no duplicate rows
- Filtered sub-counts obtained via Repository-page filters are internally consistent with the statistics-page totals

## Limitations

- Statistics reflect a specific release snapshot (MIBiG 2.0, 2019); counts change as new entries are added, so figures must be versioned and dated
- Web-UI-derived counts depend on the statistics page being current; low-quality entries may have been retired during curation, affecting totals
- Taxonomic distributions are biased toward well-studied genera (e.g. Streptomyces dominates with 568 entries) and should not be read as ecological prevalence

## Evidence

- [abstract] we improved the user experience by adding new features such as query searches and a statistics page: "We also redesigned the data schema to ensure the compliance of future annotations. Finally, we improved the user experience by adding new features such as query searches and a statistics page, and"
- [intro] fast filtering based on compound names, taxonomic identifiers or biosynthetic classes: "enabling fast filtering based on compound names, taxonomic identifiers or biosynthetic classes, and facilitating the building of Boolean queries"
- [intro] Repository page filtered and sorted by annotation metadata: "Rows in the ‘Repository’ page can now be filtered and sorted based on annotation metadata"
- [supplementary] MIBiG repository URL: "The MIBiG Repository is available at https://mibig.secondarymetabolites.org/."
- [intro] API handles access to the PostgreSQL database: "handles access to the underlying PostgreSQL"
