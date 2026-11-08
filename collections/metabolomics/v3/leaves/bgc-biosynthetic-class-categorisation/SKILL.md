---
name: bgc-biosynthetic-class-categorisation
description: Use when a BGC metadata table (e.g. the 851 new MIBiG 2.0 entries) must
  be grouped by structure-based biosynthetic class to quantify composition, dominance
  of Polyketides/NRPs, or database growth.
when_to_use_negative:
- Input is candidate BGCs predicted de novo by antiSMASH/ClusterFinder with no known
  product — MIBiG class assignment requires BGCs of known function with curated annotations
- Goal is novelty assessment of BGCs against a reference database rather than composition
  counting of the curated repository itself
- The dataset is already a per-class summary table; re-running the grouping would
  double-count hybrids
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0114
tools:
- name: MIBiG Repository
  role: Source of curated BGC annotations; rows filtered/sorted by biosynthetic class
    to build per-class counts
  repo: https://mibig.secondarymetabolites.org/
- name: antiSMASH 5
  role: Customised module sideloading MIBiG annotation files to generate individual
    BGC pages carrying the class annotations being categorised
  repo: https://github.com/mibig-secmet/mibig-api/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:693d2532a8f14ac65d94aba1113a9ad4cdb29322717f55dc65397f2e22a934b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIBiG Repository
  - antiSMASH 5
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bgc-biosynthetic-class-categorisation

## Summary

Categorises curated biosynthetic gene clusters (BGCs) of known function into MIBiG's seven structure-based classes (including hybrids and 'Other') and computes per-class counts and shares, e.g. for quantifying MIBiG 2.0's growth from 1170 to 2021 entries.

## When to use

Use when a BGC metadata table (e.g. the 851 new MIBiG 2.0 entries) must be grouped by structure-based biosynthetic class to quantify composition, dominance of Polyketides/NRPs, or database growth.

## When NOT to use

- Input is candidate BGCs predicted de novo by antiSMASH/ClusterFinder with no known product — MIBiG class assignment requires BGCs of known function with curated annotations
- Goal is novelty assessment of BGCs against a reference database rather than composition counting of the curated repository itself
- The dataset is already a per-class summary table; re-running the grouping would double-count hybrids

## Inputs

- MIBiG 2.0 BGC metadata table (851 new entries plus original 1170, with biosynthetic class annotations)
- MIBiG repository rows filtered/sorted by annotation metadata (e.g. biosynthetic class)

## Outputs

- Per-class BGC counts across the seven structure-based classes, with hybrids and 'Other' flagged (e.g. mibig_2.0_composition.csv)
- Proportional share of Polyketides and NRPs among newly added BGCs (59% combined)
- Database growth percentage relative to MIBiG 1.0 (1170 → 2021 entries, ~73% increase)

## How to apply

Load the BGC metadata table from the MIBiG repository (via the mibig-secmet/mibig-api REST endpoint over its PostgreSQL backend, or the exported table of the 851 new entries plus the original 1170). Group entries by biosynthetic class across the seven structure-based classes, explicitly flagging hybrid clusters and 'Other' exemplars rather than forcing them into a single class. Count BGCs per class (MIBiG 2.0 reference composition: 825 Polyketides, 627 NRPs, remainder RiPPs, saccharides, other, hybrids). For new-entry analysis, restrict the grouping to the 851 added clusters and compute the proportional Polyketide/NRP share (59% together). Finally, verify the total against the expected 2021 entries and express growth as (2021 − 1170) / 1170 ≈ 73%, writing per-class counts and shares to a composition CSV such as mibig_2.0_composition.csv.

## Related tools

- **MIBiG Repository** (Source of curated BGC annotations; rows filtered/sorted by biosynthetic class to build per-class counts) — https://mibig.secondarymetabolites.org/
- **antiSMASH 5** (Customised module sideloading MIBiG annotation files to generate individual BGC pages carrying the class annotations being categorised) — https://github.com/mibig-secmet/mibig-api/

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('mibig_2.0_bgcs.csv'); print(df.groupby('biosynthetic_class').size()); df.to_csv('mibig_2.0_composition.csv')"
```

## Evaluation signals

- Sum of per-class counts equals the verified total of 2021 curated BGC entries
- Polyketide count is 825 and NRP count is 627 in the full MIBiG 2.0 table
- Polyketide+NRP share among the 851 new entries equals ~59%
- Growth calculation (2021−1170)/1170 yields ~73% increase from MIBiG 1.0
- Hybrid and 'Other' entries are explicitly flagged, not silently dropped from the class counts

## Limitations

- Class assignment depends on manual curation quality; MIBiG 2.0 had to fix empty or mistyped information and remove duplicate entries, so stale exports may misclassify clusters
- Low-quality entries may be retired from the repository, shifting per-class counts between versions
- Hybrid and 'Other' categories complicate proportional shares — clusters spanning classes must be flagged rather than force-assigned

## Evidence

- [intro] 851 new entries over five years and expansion to 2021 BGCs: "we provide an updated MIBiG version 2.0, which has been significantly expanded through the addition of 851 new entries over the past five years"
- [intro] Repository rows can be filtered by biosynthetic class: "Rows in the ‘Repository’ page can now be filtered and sorted based on annotation metadata"
- [intro] Manual curation fixed mistyped data and removed duplicates: "fixing empty or mistyped information in the data, removing duplicate entries"
- [supplementary] MIBiG repository is the online source of curated BGCs: "The MIBiG Repository is available at https://mibig.secondarymetabolites.org/."
- [readme] API README identifies the served repository: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
