---
name: class-share-computation
description: Use when analyzing the composition of MIBiG 2.0's 851 newly added BGCs
  across the seven structure-based classes, or when quantifying growth of a curated
  database relative to a prior version.
when_to_use_negative:
- The dataset is not versioned against a prior baseline (e.g. candidate BGCs from
  antiSMASH-DB or IMG-ABC with no 'original vs. added' split), so no growth percentage
  can be defined
- The goal is novelty assessment of unknown BGCs by homology search against MIBiG
  (as in Crits-Christoph et al.) rather than composition statistics of curated known-function
  entries
- Entries lack biosynthetic class annotations, making per-class grouping and proportional
  shares unreliable
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MIBiG Repository
  role: Source of the curated BGC metadata and biosynthetic class annotations used
    to compute per-class counts and growth percentages
  repo: https://mibig.secondarymetabolites.org/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:cabdc3b6bfa055f80e507e1803d01ca3839eaee5d6cf9d501964be80cfbdb354
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIBiG Repository
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# class-share-computation

## Summary

Compute the proportional share of biosynthetic classes (e.g. Polyketides and NRPs) among a subset of MIBiG entries, and express repository growth as a percentage increase over a baseline count (e.g. 851 new entries vs. 1170 original = 73% growth to 2021 BGCs).

## When to use

Use when analyzing the composition of MIBiG 2.0's 851 newly added BGCs across the seven structure-based classes, or when quantifying growth of a curated database relative to a prior version.

## When NOT to use

- The dataset is not versioned against a prior baseline (e.g. candidate BGCs from antiSMASH-DB or IMG-ABC with no 'original vs. added' split), so no growth percentage can be defined
- The goal is novelty assessment of unknown BGCs by homology search against MIBiG (as in Crits-Christoph et al.) rather than composition statistics of curated known-function entries
- Entries lack biosynthetic class annotations, making per-class grouping and proportional shares unreliable

## Inputs

- MIBiG 2.0 BGC metadata table (851 new entries plus the original 1170), from https://mibig.secondarymetabolites.org/
- Biosynthetic class annotations per BGC (seven structure-based classes, including hybrids and 'Other')

## Outputs

- mibig_2.0_composition.csv with per-class BGC counts, the 59% Polyketide/NRP share of new entries, and the 73% growth figure

## How to apply

Load the MIBiG 2.0 BGC metadata table containing the 851 new entries plus the original 1170 from the MIBiG repository (https://mibig.secondarymetabolites.org/). Group entries by biosynthetic class and count BGCs in each of the seven structure-based classes, flagging hybrids and 'Other' exemplars. Compute the proportional share of Polyketides and NRPs among the 851 newly added BGCs by dividing per-class counts by the new-entry total. Verify the total entry count of 2021 and express growth as (2021 - 1170) / 1170 = 73% relative to the original release. Write per-class counts, the 59% combined Polyketide/NRP share, and the 73% increase into an output CSV such as mibig_2.0_composition.csv.

## Related tools

- **MIBiG Repository** (Source of the curated BGC metadata and biosynthetic class annotations used to compute per-class counts and growth percentages) — https://mibig.secondarymetabolites.org/

## Examples

```
python compose_stats.py --mibig-metadata mibig_2_0_entries.json --baseline 1170 --new 851 --output mibig_2.0_composition.csv
```

## Evaluation signals

- Total entry count reconciles to 2021 curated BGCs in MIBiG 2.0
- Growth percentage equals (2021 - 1170) / 1170 ≈ 73% relative to the original 1170 entries
- Combined Polyketide + NRP share among the 851 new entries equals 59%, with absolute counts of 825 Polyketides and 627 NRPs across the full database
- Per-class counts sum across all seven structure-based classes, with hybrids and 'Other' explicitly flagged
- Output CSV (mibig_2.0_composition.csv) contains one row per class with count and proportional-share columns populated

## Limitations

- Class shares depend on MIBiG curation quality; the release relied on extensive manual re-annotation, and low-quality or duplicate entries were retired, so shares can shift between versions
- Hybrid and 'Other' category assignments are exemplar-dependent and may complicate strict per-class proportioning
- The analysis describes curated BGCs of known function only; it does not represent candidate BGCs from genome mining databases

## Evidence

- [abstract] 851 new BGCs added, growth relative to 1170: "Over the past five years, 851 new BGCs have been added."
- [intro] MIBiG 2.0 update scope: "we provide an updated MIBiG version 2.0, which has been significantly expanded"
- [supplementary] Repository location for loading metadata: "The MIBiG Repository is available at https://mibig.secondarymetabolites.org/."
- [intro] Quality curation affecting counts: "we performed extensive re-annotation of the entire database, increasing the overall data quality"
