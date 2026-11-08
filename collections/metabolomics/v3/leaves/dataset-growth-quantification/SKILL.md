---
name: dataset-growth-quantification
description: Use when a database release report must state how many records were added
  since the previous version, expressed both as absolute counts (851 new BGCs) and
  as percentage growth (73% from 1170 to 2021), broken down by class.
when_to_use_negative:
- The goal is predicting or mining candidate BGCs from genomes (e.g. via antiSMASH
  or ClusterFinder) rather than summarizing a curated repository's composition
- Assessing novelty of a specific BGC against MIBiG (as in homology-search use cases)
  — this skill aggregates repository metadata, not individual cluster comparisons
- The dataset has no versioned baseline to compare against, so a growth percentage
  cannot be defined
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MIBiG Repository
  role: Source of the curated BGC metadata table and per-entry biosynthetic class
    annotations used for counting and growth computation
  repo: https://mibig.secondarymetabolites.org/
- name: MIBiG API
  role: REST-like web API providing programmatic access to the underlying PostgreSQL
    database holding the MIBiG entries being counted
  repo: https://github.com/mibig-secmet/mibig-api/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:2da57bd8b0498f8b1bce5f0708adcb6dc8e2bc3aad164871324bb15246e4c908
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIBiG API
  - MIBiG Repository
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# dataset-growth-quantification

## Summary

Quantifies the expansion of a curated reference dataset between versions by loading its metadata table, counting entries per category, and computing proportional shares and percentage growth. Applied here to MIBiG 2.0, which grew from 1170 to 2021 curated BGCs (73% increase) across seven structure-based biosynthetic classes including hybrids.

## When to use

Use when a database release report must state how many records were added since the previous version, expressed both as absolute counts (851 new BGCs) and as percentage growth (73% from 1170 to 2021), broken down by class.

## When NOT to use

- The goal is predicting or mining candidate BGCs from genomes (e.g. via antiSMASH or ClusterFinder) rather than summarizing a curated repository's composition
- Assessing novelty of a specific BGC against MIBiG (as in homology-search use cases) — this skill aggregates repository metadata, not individual cluster comparisons
- The dataset has no versioned baseline to compare against, so a growth percentage cannot be defined

## Inputs

- MIBiG 2.0 BGC metadata table (2021 entries, including the 851 added since MIBiG 1.0) from https://mibig.secondarymetabolites.org/
- Class annotation per BGC (structure-based biosynthetic class: Polyketide, NRP, RiPP, etc., including hybrid designations)
- Baseline version entry count (MIBiG 1.0: 1170 BGCs)

## Outputs

- mibig_2.0_composition.csv with per-class BGC counts, the 59% Polyketide/NRP share among new entries, and the 73% growth figure
- Verified total entry count (2021) and percentage increase relative to 1170

## How to apply

Load the dataset's metadata table for all entries — here the MIBiG 2.0 table of 2021 BGCs, of which 851 were added since the 2015 release. Group entries by the dataset's own category field (the seven structure-based biosynthetic classes, flagging hybrids and 'Other' exemplars) and count records per class. Compute the proportional share of dominant categories among the NEW entries only (Polyketides + NRPs together make up 59% of the 851 additions), then verify the total against the published count and express growth as (new - old) / old x 100, i.e. (2021 - 1170) / 1170 = 73%. Before counting, ensure entries have passed the repository's curation checks — fixing empty or mistyped information, removing duplicate entries, and retiring low-quality entries — so the growth figure reflects curated rather than raw records. Write per-class counts, the new-entry class shares, and the growth percentage to a summary CSV (e.g. mibig_2.0_composition.csv).

## Related tools

- **MIBiG Repository** (Source of the curated BGC metadata table and per-entry biosynthetic class annotations used for counting and growth computation) — https://mibig.secondarymetabolites.org/
- **MIBiG API** (REST-like web API providing programmatic access to the underlying PostgreSQL database holding the MIBiG entries being counted) — https://github.com/mibig-secmet/mibig-api/

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('mibig_bgcs.csv'); counts=df[df['is_new']].groupby('biosynthetic_class').size(); print(((2021-1170)/1170)*100); counts.to_csv('mibig_2.0_composition.csv')"
```

## Evaluation signals

- Per-class counts sum to the verified total of 2021 BGCs, including hybrids and 'Other' entries
- Growth percentage reproduces (2021 - 1170) / 1170 = 73% when recomputed from raw counts
- Polyketide (825) and NRP (627) counts, and their combined share, reproduce the reported dominance of these two classes
- Share of Polyketides + NRPs among the 851 new entries alone equals 59% (distinct from their share of all 2021 entries)
- Counted entries exclude duplicates and retired low-quality records, matching the repository's curation pass

## Limitations

- Counts depend on extensive manual curation ('Annotathons' and re-annotation of the entire database); entries retired as low-quality or de-duplicated shift the totals between versions
- Hybrid BGCs and the 'Other' category make per-class shares non-exclusive unless a hybrid handling rule (primary class vs. multi-class flag) is fixed before counting
- Growth figures are specific to the 2015-to-2019 comparison window (851 additions over five years); they do not generalize to other update intervals without re-counting

## Evidence

- [abstract] 851 new BGCs added: "Over the past five years, 851 new BGCs have been added."
- [intro] MIBiG 2.0 expansion: "we provide an updated MIBiG version 2.0, which has been significantly expanded through the addition of 851 new entries over the past five years"
- [intro] curation affects counts: "fixing empty or mistyped information in the data, removing duplicate entries"
- [intro] filtering by biosynthetic class: "enabling fast filtering based on compound names, taxonomic identifiers or biosynthetic classes, and facilitating the building of Boolean queries"
- [readme] repository access point: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
