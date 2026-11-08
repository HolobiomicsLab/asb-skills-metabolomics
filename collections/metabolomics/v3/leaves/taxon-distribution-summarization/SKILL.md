---
name: taxon-distribution-summarization
description: Use when a question asks for headline counts of a curated BGC repository
  (total entries, per-class breakdown such as Polyketide/NRP/RiPP/'Other', or per-taxon
  counts like Streptomyces/Aspergillus) rather than novel cluster prediction.
when_to_use_negative:
- The goal is detecting or annotating candidate BGCs from genome/metagenome assemblies
  (use antiSMASH-type genome mining, not repository statistics extraction)
- The target is databases of many thousands of uncharacterized/candidate clusters
  such as IMG-ABC or antiSMASH-DB — this skill counts only curated BGCs of known chemical
  structure in MIBiG
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MIBiG Repository
  role: Web interface whose statistics page and annotation-metadata filters supply
    the entry counts, class distributions, and taxonomic distributions to be tabulated
  repo: https://mibig.secondarymetabolites.org/
- name: MIBiG API
  role: REST-like programmatic access to the underlying PostgreSQL database, an alternative
    route to reconstruct class/taxon counts
  repo: github.com/mibig-secmet/mibig-api
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:897534191ce098bac28a00fbda880e199cafff22336ee8ea25d0e8c280c31dfd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIBiG API
  - MIBiG Repository
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# taxon-distribution-summarization

## Summary

Extract and tabulate curated-BGC counts and their biosynthetic-class and taxonomic distributions from the MIBiG repository statistics page, producing a structured summary table that reconciles repository totals (e.g. MIBiG 2.0: 2021 curated BGCs) against the published article.

## When to use

Use when a question asks for headline counts of a curated BGC repository (total entries, per-class breakdown such as Polyketide/NRP/RiPP/'Other', or per-taxon counts like Streptomyces/Aspergillus) rather than novel cluster prediction.

## When NOT to use

- The goal is detecting or annotating candidate BGCs from genome/metagenome assemblies (use antiSMASH-type genome mining, not repository statistics extraction)
- The target is databases of many thousands of uncharacterized/candidate clusters such as IMG-ABC or antiSMASH-DB — this skill counts only curated BGCs of known chemical structure in MIBiG

## Inputs

- MIBiG repository statistics page (https://mibig.secondarymetabolites.org/)
- MIBiG REST-like web API (github.com/mibig-secmet/mibig-api) responses
- Repository rows filtered by taxonomic identifiers or biosynthetic classes

## Outputs

- Structured summary table (e.g. mibig2_summary_stats.csv) with columns: statistic category (total/class/taxon), label, count
- Reconciled set of headline statistics: total curated BGC count, per-class distribution, per-taxon distribution

## How to apply

Navigate to the repository statistics page at https://mibig.secondarymetabolites.org/ and read the published total count of curated entries, per-class counts (polyketide, NRP, hybrid classes, and 'Other'), and per-taxon counts. Alternatively, query the REST-like web API behind the relational PostgreSQL database, or use the repository page's filtering based on compound names, taxonomic identifiers or biosynthetic classes, to reconstruct the same distributions. Tabulate values into a long-format table with columns for statistic category (total/class/taxon), label, and count. Decision point: only entries of known function belong in these counts — databases like IMG-ABC and antiSMASH-DB store many thousands of candidate BGCs and must not be mixed in. Cross-check extracted values against figures reported in the source article (for MIBiG 2.0: 2021 total; Polyketide 825; NRP 627; Streptomyces 568, Aspergillus 79, Pseudomonas 61; 19 plant BGCs) and save as a summary CSV.

## Related tools

- **MIBiG Repository** (Web interface whose statistics page and annotation-metadata filters supply the entry counts, class distributions, and taxonomic distributions to be tabulated) — https://mibig.secondarymetabolites.org/
- **MIBiG API** (REST-like programmatic access to the underlying PostgreSQL database, an alternative route to reconstruct class/taxon counts) — github.com/mibig-secmet/mibig-api

## Examples

```
python -c "import pandas as pd; df=pd.DataFrame({'category':['total','class','class','taxon','taxon','taxon'],'label':['curated_BGCs','Polyketide','NRP','Streptomyces','Aspergillus','Pseudomonas'],'count':[2021,825,627,568,79,61]}); df.to_csv('mibig2_summary_stats.csv',index=False)"
```

## Evaluation signals

- Class counts must sum to the total curated entry count (MIBiG 2.0: 2021; Polyketide 825 + NRP 627 + hybrid/other classes = 2021)
- Taxon counts must match reported top genera: Streptomyces 568, Aspergillus 79, Pseudomonas 61, plants 19
- Every row in the output table carries a valid statistic category (total/class/taxon), a label, and a non-negative integer count
- Extracted values agree with the figures reported in the MIBiG 2.0 article text

## Limitations

- Counts reflect a specific repository snapshot (MIBiG 2.0 with 2021 entries, having grown by 851 entries over five years); future versions will shift distributions
- No changelog was available for tracking historical count changes, so longitudinal comparisons cannot be reconstructed from the repository alone
- Values depend on manual curation quality; low-quality or duplicate entries are periodically retired, altering counts

## Evidence

- [intro] we provide an updated MIBiG version 2.0, which has been significantly expanded through the addition of 851 new entries over the past five years: "we provide an updated MIBiG version 2.0, which has been significantly expanded through the addition of 851 new entries over the past five years"
- [intro] Rows in the 'Repository' page can now be filtered and sorted based on annotation metadata: "Rows in the ‘Repository’ page can now be filtered and sorted based on annotation metadata"
- [supplementary] The MIBiG Repository is available at https://mibig.secondarymetabolites.org/.: "The MIBiG Repository is available at https://mibig.secondarymetabolites.org/."
- [readme] The WebAPI server driving the MIBiG repository.: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
