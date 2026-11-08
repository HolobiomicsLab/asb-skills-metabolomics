---
name: repository-composition-statistics
description: Use when a curated reference database (e.g. MIBiG) has released a new
  version and you must report how many entries were added, how they distribute across
  predefined classes (including hybrids and 'Other'), and the proportional share of
  dominant classes.
when_to_use_negative:
- Input is raw candidate BGC predictions from genome mining (e.g. thousands of candidates
  from antiSMASH or IMG-ABC/antiSMASH-DB) rather than curated BGCs of known function
  — MIBiG counts only validated, literature-backed entries
- The goal is to submit or deposit a new BGC entry to MIBiG (accession request, entry
  form, Annotathon curation) rather than to compute repository-level statistics
- You need per-entry annotation curation (fixing mistyped info, removing duplicates,
  adding chemical structures) rather than aggregate composition metrics
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3366
tools:
- name: MIBiG Repository
  role: Source of the curated BGC metadata table and online filtering/query features
    used to group entries by biosynthetic class and verify counts
  repo: https://mibig.secondarymetabolites.org/
- name: mibig-api
  role: REST-like web API providing programmatic access to the underlying MIBiG PostgreSQL
    database for retrieving entry-level metadata
  repo: https://github.com/mibig-secmet/mibig-api/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:0936c23a87989cba1df3a1d921faa4f53d72b21e1e012b05b8e24db6f6618d6d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIBiG Repository
  - mibig-api
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# repository-composition-statistics

## Summary

Quantifies the composition and growth of the MIBiG 2.0 repository by counting curated biosynthetic gene clusters (BGCs) per structure-based class and computing growth relative to a baseline release. It turns repository metadata into per-class counts and percentage-increase figures suitable for reporting database expansion.

## When to use

Use when a curated reference database (e.g. MIBiG) has released a new version and you must report how many entries were added, how they distribute across predefined classes (including hybrids and 'Other'), and the proportional share of dominant classes.

## When NOT to use

- Input is raw candidate BGC predictions from genome mining (e.g. thousands of candidates from antiSMASH or IMG-ABC/antiSMASH-DB) rather than curated BGCs of known function — MIBiG counts only validated, literature-backed entries
- The goal is to submit or deposit a new BGC entry to MIBiG (accession request, entry form, Annotathon curation) rather than to compute repository-level statistics
- You need per-entry annotation curation (fixing mistyped info, removing duplicates, adding chemical structures) rather than aggregate composition metrics

## Inputs

- MIBiG 2.0 BGC metadata table (851 new entries plus the original 1170 entries), from https://mibig.secondarymetabolites.org/
- Biosynthetic class annotation per BGC entry (seven structure-based classes, including hybrids and 'Other')
- Baseline entry count from the previous release (MIBiG 1.0: 1170 entries)

## Outputs

- Per-class BGC counts for the seven structure-based classes, with hybrids and 'Other' exemplars flagged
- Proportional share of Polyketides and NRPs among newly added BGCs (59% combined for the 851 new entries)
- Overall growth percentage relative to baseline (2021 vs 1170 entries, ~73% increase), e.g. written to mibig_2.0_composition.csv

## How to apply

Load the MIBiG 2.0 BGC metadata table containing the 851 new entries plus the original 1170 entries. Group entries by biosynthetic class and count BGCs in each of the seven structure-based classes, explicitly flagging hybrid clusters and 'Other' exemplars; expect Polyketides (825) and NRPs (627) to dominate. Compute the proportional share of Polyketides and NRPs among the 851 newly added entries (they jointly account for ~59% of new entries). Verify the total curated entry count is 2021 and express growth as a percentage increase relative to the original 1170 entries, yielding ~73%. Use the repository's filtering/query features (Boolean queries on biosynthetic classes, compound names, taxonomic identifiers) to cross-check per-class counts before writing results to a composition table such as mibig_2.0_composition.csv.

## Related tools

- **MIBiG Repository** (Source of the curated BGC metadata table and online filtering/query features used to group entries by biosynthetic class and verify counts) — https://mibig.secondarymetabolites.org/
- **mibig-api** (REST-like web API providing programmatic access to the underlying MIBiG PostgreSQL database for retrieving entry-level metadata) — https://github.com/mibig-secmet/mibig-api/

## Examples

```
import pandas as pd; df = pd.read_csv('mibig_2.0_bgc_metadata.csv'); counts = df.groupby('biosynthetic_class').size(); growth = round((len(df) - 1170) / 1170 * 100, 1)
```

## Evaluation signals

- Total entry count reconciles to 2021 curated BGCs; adding 851 new entries to the 1170 baseline reproduces the reported total exactly
- Growth percentage computes to ~73%: (2021 - 1170) / 1170
- Polyketide count (825) and NRP count (627) match reported class totals, and their combined share of the 851 new entries is ~59%
- Per-class counts are mutually exclusive of the seven structure-based classes with hybrids and 'Other' explicitly flagged (no double counting)
- Output composition table (e.g. mibig_2.0_composition.csv) contains every class with a non-negative integer count summing to the verified total

## Limitations

- Counts are limited to BGCs of known function; candidate BGCs stored in databases like IMG-ABC and antiSMASH-DB are out of scope and cannot be compared 1:1
- The 'Other' category and hybrid clusters make class-wise shares less interpretable than single-class counts
- The README provides no changelog, so version-to-version diffs must be reconstructed from the article's stated figures (851 added, 1170 → 2021) rather than a release history

## Evidence

- [abstract] Over the past five years, 851 new BGCs have been added.: "Over the past five years, 851 new BGCs have been added. Additionally, we performed extensive manual data curation of all entries"
- [intro] Updated MIBiG version 2.0 expanded through 851 new entries: "we provide an updated MIBiG version 2.0, which has been significantly expanded through the addition of 851 new entries over the past five years"
- [intro] Fast filtering on biosynthetic classes to cross-check counts: "enabling fast filtering based on compound names, taxonomic identifiers or biosynthetic classes, and facilitating the building of Boolean queries"
- [supplementary] MIBiG repository access URL: "The MIBiG Repository is available at https://mibig.secondarymetabolites.org/."
