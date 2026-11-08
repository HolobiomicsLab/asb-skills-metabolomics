---
name: bgc-repository-statistics-extraction
description: Use when you need aggregate counts of curated BGCs — totals, per-class
  (e.g. Polyketide 825, NRP 627) or per-taxon (e.g. Streptomyces 568) breakdowns —
  from MIBiG or a similar curated BGC repository.
when_to_use_negative:
- When you need individual BGC entry records or sequences rather than aggregate statistics
  — use the repository's per-entry pages or the REST-like API instead
- When counting candidate (uncurated) BGCs predicted by tools like antiSMASH or ClusterFinder
  — MIBiG statistics cover only manually curated BGCs of known function
- When you need programmatically queryable bulk data rather than summary counts —
  use the mibig-api GitHub endpoint or JSON-formatted data standard
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_3314
tools:
- name: MIBiG Repository
  role: Source of the statistics page and filterable repository rows from which entry,
    class, and taxonomy counts are extracted
  repo: https://mibig.secondarymetabolites.org/
- name: GitHub
  role: Hosts the source code, JSON-formatted data standard, and SQL schema underlying
    the repository, useful for validating extracted counts
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:f3e9440373059b0afd0d36b0e60cebcdc0908f6a975a6efac376bc3e91a65162
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - MIBiG Repository
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bgc-repository-statistics-extraction

## Summary

Extracts and tabulates curated-entry counts, biosynthetic class distributions, and taxonomic distributions from the MIBiG repository's statistics page (version 2.0: 2021 BGCs), cross-checking them against the published article figures.

## When to use

Use when you need aggregate counts of curated BGCs — totals, per-class (e.g. Polyketide 825, NRP 627) or per-taxon (e.g. Streptomyces 568) breakdowns — from MIBiG or a similar curated BGC repository.

## When NOT to use

- When you need individual BGC entry records or sequences rather than aggregate statistics — use the repository's per-entry pages or the REST-like API instead
- When counting candidate (uncurated) BGCs predicted by tools like antiSMASH or ClusterFinder — MIBiG statistics cover only manually curated BGCs of known function
- When you need programmatically queryable bulk data rather than summary counts — use the mibig-api GitHub endpoint or JSON-formatted data standard

## Inputs

- MIBiG repository statistics page (https://mibig.secondarymetabolites.org/)
- MIBiG 2.0 article (DOI: 10.1093/nar/gkz882) for cross-checking reported counts

## Outputs

- Summary CSV (e.g. mibig2_summary_stats.csv) with columns: statistic category (total/class/taxon), label, count
- Tabulated counts: 2021 total BGCs; per-class (Polyketide 825, NRP 627, hybrids, Other); per-taxon (Streptomyces 568, Aspergillus 79, Pseudomonas 61, plants 19)

## How to apply

Access the MIBiG repository statistics page at https://mibig.secondarymetabolites.org/, which was introduced in MIBiG 2.0 specifically to publish entry, class, and taxonomy counts. Extract the total manually curated BGC count (2021 in version 2.0, expanded from 1170 with 851 new entries over five years), per-class counts including Polyketide, NRP, hybrids, and 'Other', and per-taxon counts. Tabulate into a structured CSV with columns for statistic category (total/class/taxon), label, and count. Cross-check every extracted value against the figures reported in the MIBiG 2.0 Nucleic Acids Research article before saving (e.g. as mibig2_summary_stats.csv). If finer-grained counts are needed, use the repository's filtering: rows on the 'Repository' page can be filtered and sorted based on annotation metadata such as species names or biosynthetic classes.

## Related tools

- **MIBiG Repository** (Source of the statistics page and filterable repository rows from which entry, class, and taxonomy counts are extracted) — https://mibig.secondarymetabolites.org/
- **GitHub** (Hosts the source code, JSON-formatted data standard, and SQL schema underlying the repository, useful for validating extracted counts) — https://github.com/mibig-secmet

## Examples

```
curl -s https://mibig.secondarymetabolites.org/api/v1/stats && python tabulate_stats.py --source stats.json --categories total,class,taxon --out mibig2_summary_stats.csv
```

## Evaluation signals

- Total curated BGC count equals 2021 for MIBiG 2.0 (matches article: expanded from 1170 with 851 new entries)
- Per-class counts sum consistently with the total: Polyketide 825, NRP 627, hybrid classes, and 'Other' account for all entries
- Per-taxon counts match article-reported values: Streptomyces 568, Aspergillus 79, Pseudomonas 61, plants 19
- Output CSV contains exactly three columns (statistic category, label, count) with no missing or duplicated rows
- Every extracted value has a corroborating figure in the MIBiG 2.0 article text

## Limitations

- Counts reflect the repository snapshot at retrieval time; MIBiG grows via crowdsourcing, 'Annotathons', and classroom curation, so values will drift after version 2.0
- Statistics reflect manually curated entries only; low-quality entries may be retired and duplicates removed during curation, shifting counts between snapshots
- The paper notes 'No changelog found' in the discussion, so version-to-version count provenance may be incomplete

## Evidence

- [abstract] We also redesigned the data schema to ensure the compliance of future annotations. Finally, we improved the user experience by adding new features such as query searches and a statistics page: "we improved the user experience by adding new features such as query searches and a statistics page"
- [intro] intro expansion quote: "we provide an updated MIBiG version 2.0, which has been significantly expanded through the addition of 851 new entries over the past five years"
- [intro] filtering quote: "Rows in the ‘Repository’ page can now be filtered and sorted based on annotation metadata"
- [readme] readme quote: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
