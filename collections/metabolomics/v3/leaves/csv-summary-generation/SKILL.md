---
name: csv-summary-generation
description: Use when a repository/dataset metadata table must be summarized into
  a small CSV with per-category counts, proportional shares of new entries, and percent
  growth versus a prior version.
when_to_use_negative:
- Input is raw genome/sequence data rather than an already-annotated BGC metadata
  table with biosynthetic class labels
- The goal is BGC prediction or detection (e.g. antiSMASH analysis) rather than summarizing
  counts of already-curated entries
- Counts must be derived from primary literature curation rather than the repository's
  own metadata, since MIBiG numbers reflect its manual curation state
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3366
tools:
- name: MIBiG Repository
  role: Source of the BGC metadata table whose entries are grouped by the seven structure-based
    classes and counted for the summary CSV
  repo: https://mibig.secondarymetabolites.org/
- name: mibig-api
  role: REST-like web API providing programmatic access to the underlying PostgreSQL
    database holding the MIBiG entries used for the counts
  repo: https://github.com/mibig-secmet/mibig-api/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:15d7a8cd6f3e175dbc239b0d56926ba826eefbd4e567660bf6a35604a8aac0a0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIBiG Repository
  - mibig-api
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# csv-summary-generation

## Summary

Generates a per-class composition summary CSV (e.g. mibig_2.0_composition.csv) from a curated BGC metadata table, computing class counts, class proportions among newly added entries, and growth percentages relative to a baseline release.

## When to use

Use when a repository/dataset metadata table must be summarized into a small CSV with per-category counts, proportional shares of new entries, and percent growth versus a prior version.

## When NOT to use

- Input is raw genome/sequence data rather than an already-annotated BGC metadata table with biosynthetic class labels
- The goal is BGC prediction or detection (e.g. antiSMASH analysis) rather than summarizing counts of already-curated entries
- Counts must be derived from primary literature curation rather than the repository's own metadata, since MIBiG numbers reflect its manual curation state

## Inputs

- MIBiG 2.0 BGC metadata table (851 new entries plus the original 1170 entries)
- MIBiG repository export with biosynthetic class annotations per BGC

## Outputs

- mibig_2.0_composition.csv with per-class counts, 59% Polyketide/NRP share among new entries, and 73% growth figure

## How to apply

Load the MIBiG 2.0 BGC metadata table (851 new entries plus the original 1170) from the MIBiG Repository dataset. Group entries by biosynthetic class and count BGCs in each of the seven structure-based classes, flagging hybrids and 'Other' exemplars (Polyketides 825, NRPs 627 dominate). Compute the proportional share of Polyketides and NRPs among the 851 newly added BGCs (59% combined). Verify the total entry count (2021) and express growth as (2021 - 1170) / 1170 ≈ 73%. Write the per-class counts, the 59% Polyketide/NRP share, and the 73% increase into mibig_2.0_composition.csv.

## Related tools

- **MIBiG Repository** (Source of the BGC metadata table whose entries are grouped by the seven structure-based classes and counted for the summary CSV) — https://mibig.secondarymetabolites.org/
- **mibig-api** (REST-like web API providing programmatic access to the underlying PostgreSQL database holding the MIBiG entries used for the counts) — https://github.com/mibig-secmet/mibig-api/

## Examples

```
import pandas as pd; df = pd.read_csv('mibig_2.0_bgc_metadata.csv'); counts = df['biosynthetic_class'].value_counts(); new_share = df[df['is_new']]['biosynthetic_class'].value_counts(normalize=True)[['Polyketide','NRP']].sum(); counts.append({'total_growth_pct': (len(df)-1170)/1170*100}).to_csv('mibig_2.0_composition.csv')
```

## Evaluation signals

- Total verified entry count equals 2021 BGCs (1170 original + 851 new)
- Growth percentage computed as (2021 - 1170)/1170 ≈ 73%
- Combined Polyketide + NRP share among the 851 new entries equals 59%; absolute counts 825 Polyketides and 627 NRPs
- Exactly seven structure-based classes represented, with hybrids and 'Other' exemplars flagged
- Output CSV contains rows for every class plus the aggregate growth metric and parses as valid tabular data

## Limitations

- Counts are only as accurate as MIBiG's manual curation; the article notes extensive re-annotation was needed to fix 'empty or mistyped information' and remove duplicate entries
- Hybrid and 'Other' class assignments can complicate unambiguous per-class counting
- No changelog was available, so version-to-version diffing of entry sets must rely on the repository metadata itself

## Evidence

- [abstract] Over the past five years, 851 new BGCs have been added.: "Over the past five years, 851 new BGCs have been added."
- [intro] Updated MIBiG version 2.0 with 851 new entries over five years: "we provide an updated MIBiG version 2.0, which has been significantly expanded through the addition of 851 new entries over the past five years"
- [supplementary] Repository location: "The MIBiG Repository is available at https://mibig.secondarymetabolites.org/."
- [intro] Filtering by biosynthetic classes: "enabling fast filtering based on compound names, taxonomic identifiers or biosynthetic classes"
