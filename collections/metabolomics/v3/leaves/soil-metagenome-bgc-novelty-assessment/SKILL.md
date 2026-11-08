---
name: soil-metagenome-bgc-novelty-assessment
description: Use when metagenomic or uncultivated-bacterial BGC predictions need to
  be compared against a curated reference set of known-function BGCs (MIBiG 2.0) to
  quantify novelty or correlate BGC abundance with other genomic features.
when_to_use_negative:
- When the goal is de novo BGC detection rather than novelty assessment against known
  clusters — use antiSMASH/ClusterFinder mining directly instead of the MIBiG homology
  baseline
- When the query BGCs are already annotated with known function and deposited in MIBiG;
  the novelty comparison is circular for curated entries
- When comparing against general-purpose candidate-BGC databases such as IMG-ABC or
  antiSMASH-DB, which store many thousands of unvalidated candidate BGCs rather than
  BGCs of known function
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: MIBiG Repository
  role: Reference corpus of BGCs of known function used as the homology baseline for
    novelty and association analyses
  repo: https://mibig.secondarymetabolites.org/
- name: mibig-api
  role: REST-like web API for programmatic access to the underlying PostgreSQL database
    of MIBiG entries
  repo: https://github.com/mibig-secmet/mibig-api/
- name: antiSMASH
  role: Genome-mining tool for detecting BGCs that are then queried against MIBiG;
    MIBiG BGC pages are generated with a customised antiSMASH 5 module that sideloads
    a MIBiG annotation file
- name: ClusterCAD
  role: Computer-aided pathway design tool that sources BGC data from MIBiG, useful
    for downstream design of novel pathways from novelty-assessed clusters
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:366717d814541e0887fd1f0238b5604dcd884c62560c6686fbd01cbdc324e575
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClusterCAD
  - GitHub
  - MIBiG Repository
  - PostgreSQL
  - antiSMASH
  - mibig-api
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# soil-metagenome-bgc-novelty-assessment

## Summary

A method for assessing how novel soil-metagenome-derived Biosynthetic Gene Clusters (BGCs) are by using MIBiG as the homology baseline, as demonstrated by Crits-Cristoph et al. (uncultivated soil bacteria) and Bahram et al. (fungal antibacterial BGCs).

## When to use

Use when metagenomic or uncultivated-bacterial BGC predictions need to be compared against a curated reference set of known-function BGCs (MIBiG 2.0) to quantify novelty or correlate BGC abundance with other genomic features.

## When NOT to use

- When the goal is de novo BGC detection rather than novelty assessment against known clusters — use antiSMASH/ClusterFinder mining directly instead of the MIBiG homology baseline
- When the query BGCs are already annotated with known function and deposited in MIBiG; the novelty comparison is circular for curated entries
- When comparing against general-purpose candidate-BGC databases such as IMG-ABC or antiSMASH-DB, which store many thousands of unvalidated candidate BGCs rather than BGCs of known function

## Inputs

- MIBiG 2.0 reference set (JSON-formatted BGC entries with biosynthetic class, taxonomy, and product annotations)
- BGC predictions from uncultivated soil-bacterial genomes or soil metagenome assemblies
- Metagenomic fungal BGC predictions
- Antimicrobial resistance gene counts across soil metagenomic samples

## Outputs

- Novelty fraction: proportion of query BGCs lacking homology to any known MIBiG cluster
- Per-BGC homology hits against MIBiG entries (with biosynthetic class and product annotations)
- Correlation coefficient between antibacterial BGC abundance and antimicrobial resistance gene counts across soil samples
- Structured CSV table collating reproduction metrics

## How to apply

Download the MIBiG 2.0 data from the official GitHub repository (https://github.com/mibig-secmet) or query it programmatically through the REST-like MIBiG API (https://github.com/mibig-secmet/mibig-api/), which fronts the underlying PostgreSQL database; parse the JSON-formatted entries to extract biosynthetic class, taxonomic identifier, and product annotation fields. For the novelty analysis, run homology searches of BGCs recovered from uncultivated soil bacteria (e.g. via antiSMASH-predicted clusters) against the MIBiG reference set and compute the fraction of clusters lacking homology to any known MIBiG entry. For the association analysis, perform homology searches of metagenomic fungal BGCs against MIBiG to identify fungal BGCs associated with antibacterial activity, then correlate antibacterial BGC abundance with antimicrobial resistance gene counts across soil samples. Optionally confirm that ClusterCAD sources BGC sequence and annotation data directly from MIBiG if the output feeds computer-aided pathway design. Collate the novelty fraction and correlation coefficient per sample/condition into a structured table and compare against the ranges reported in the original publications.

## Related tools

- **MIBiG Repository** (Reference corpus of BGCs of known function used as the homology baseline for novelty and association analyses) — https://mibig.secondarymetabolites.org/
- **mibig-api** (REST-like web API for programmatic access to the underlying PostgreSQL database of MIBiG entries) — https://github.com/mibig-secmet/mibig-api/
- **antiSMASH** (Genome-mining tool for detecting BGCs that are then queried against MIBiG; MIBiG BGC pages are generated with a customised antiSMASH 5 module that sideloads a MIBiG annotation file)
- **ClusterCAD** (Computer-aided pathway design tool that sources BGC data from MIBiG, useful for downstream design of novel pathways from novelty-assessed clusters)

## Examples

```
git clone https://github.com/mibig-secmet/mibig-api && go run server.go
```

## Evaluation signals

- Reproduced novelty fraction (fraction of uncultivated soil-bacterial BGCs lacking MIBiG homology) falls within the range reported by Crits-Cristoph et al.
- Correlation coefficient between antibacterial BGC abundance and antimicrobial resistance gene counts across soils falls within the range reported by Bahram et al.
- All MIBiG entries parsed are part of the 2021 curated BGCs in MIBiG 2.0 (expanded from 1170), confirming the correct reference version is used
- Every query BGC in the output CSV carries a homology verdict (hit or no-hit) against MIBiG, with biosynthetic class and taxonomic fields populated
- API/JSON data fetched from the mibig-api or GitHub validates against the MIBiG JSON schema description

## Limitations

- Novelty is defined relative to MIBiG coverage; BGCs similar to newly curated clusters added after the analysis date will appear less novel as the repository grows (851 new BGCs were added over five years)
- MIBiG only contains BGCs associated with molecules of known chemical structure, so homology to MIBiG is a conservative novelty measure compared with candidate-BGC databases like IMG-ABC or antiSMASH-DB
- The mibig-api server requires a recent version of Go (1.11.x) for dependency handling if self-hosting the API

## Evidence

- [intro] Crits-Cristoph et al. used MIBiG as homology baseline for soil bacteria: "Crits-Cristoph et al. (7) recently used MIBiG to assess and highlight the exceptional novelty of BGCs"
- [intro] Bahram et al. identified fungal antibacterial BGCs via MIBiG homology searches: "Bahram et al. (8) used homology searches against MIBiG to identify fungal BGCs associated with antibacterial activity"
- [intro] MIBiG is the central reference database for BGCs of known function: "the MIBiG repository has become a central reference database for BGCs of known function"
- [readme] mibig-api is the web server driving the MIBiG repository: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
- [abstract] MIBiG 2.0 scope: curated BGCs of known chemical structure: "The MIBiG (Minimum Information about a Biosynthetic Gene Cluster) Data Standard and Repository was established in 2015 to enable curation and storage of known BGCs"
