---
name: deposited-dataset-retrieval
description: Use when a headline figure reported in a paper (e.g. LOTUS's 750,000+
  referenced structure-organism pairs) must be independently verified by counting
  records in the deposited, versioned dataset file rather than trusting the manuscript
  text.
when_to_use_negative:
- When the goal is to query live, community-editable data rather than a frozen snapshot
  — LOTUS is also hosted on Wikidata with SPARQL access, which changes over time
- When the input is the uncurated initial table (2.5M+ pairs) rather than the post-validation
  3_curated table, since counts will not match the reported retained total
- When the goal is to re-run the curation/harmonization pipeline from raw sources
  instead of reading its curated output
edam_operation: http://edamontology.org/operation_0226
tools:
- name: lotus-processor
  role: Source repository whose processing pipeline produced the deposited curated
    table being retrieved and counted
  repo: https://github.com/lotusnprod/lotus-processor
- name: pandas
  role: Loads the gzip-compressed TSV (sep='\t', compression='gzip') and computes
    unique pair/structure/organism counts via duplicate dropping
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:eae73abd324e847d7166c7e0c61f18aaf98e4855bb4824c5a58748663829a0e7
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-processor
  tools:
  - lotus-processor
  - pandas
  edam_operation: http://edamontology.org/operation_0226
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# deposited-dataset-retrieval

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Retrieves and re-analyses a curated dataset table from the authors' Zenodo deposit (LOTUS lotus-processor, DOI 10.5281/zenodo.5607264) to independently reproduce published summary statistics, specifically the count of unique referenced structure-organism pairs at full stereochemical (3D) and flat (2D) levels.

## When to use

Use when a headline figure reported in a paper (e.g. LOTUS's 750,000+ referenced structure-organism pairs) must be independently verified by counting records in the deposited, versioned dataset file rather than trusting the manuscript text.

## When NOT to use

- When the goal is to query live, community-editable data rather than a frozen snapshot — LOTUS is also hosted on Wikidata with SPARQL access, which changes over time
- When the input is the uncurated initial table (2.5M+ pairs) rather than the post-validation 3_curated table, since counts will not match the reported retained total
- When the goal is to re-run the curation/harmonization pipeline from raw sources instead of reading its curated output

## Inputs

- Gzip-compressed TSV table interim/tables/3_curated/table.tsv.gz from Zenodo deposit 10.5281/zenodo.5607264
- Reported target figures from the article (588694 3D pairs, 484174 2D pairs, 750,000+ retained of 2.5M+ initial)

## Outputs

- structure_organism_pair_counts.json containing unique 3D pair count, 2D pair count, unique structure count, and unique organism count
- Verification verdict on whether the deposited table reproduces the reported dissemination totals

## How to apply

Download the lotus-processor Zenodo deposit identified by DOI 10.5281/zenodo.5607264 and locate the curated table at interim/tables/3_curated/table.tsv.gz. Load the gzip-compressed TSV into pandas with pd.read_csv(path, sep='\t', compression='gzip'). Count distinct referenced structure-organism pairs by dropping duplicate rows twice: once at the full stereochemical (3D) structure level and once at the flat (2D) level. Cross-check by separately tabulating the number of unique curated structures and unique contributing organisms. Write all four counts (3D pairs, 2D pairs, structures, organisms) to a JSON file (structure_organism_pair_counts.json) so the reproduction is machine-checkable against the reported figures 588694 (3D) and 484174 (2D), whose total is consistent with the reported retention of 750,000+ pairs (~30% of 2.5M+ initial pairs).

## Related tools

- **lotus-processor** (Source repository whose processing pipeline produced the deposited curated table being retrieved and counted) — https://github.com/lotusnprod/lotus-processor
- **pandas** (Loads the gzip-compressed TSV (sep='\t', compression='gzip') and computes unique pair/structure/organism counts via duplicate dropping)

## Examples

```
df = pd.read_csv('interim/tables/3_curated/table.tsv.gz', sep='\t', compression='gzip')
```

## Evaluation signals

- Unique 3D referenced structure-organism pair count equals 588694
- Unique 2D referenced structure-organism pair count equals 484174
- Retained pair total is consistent with 750,000+ pairs, i.e. approximately 30% of the 2.5M+ initial pairs
- Structure and organism cross-check counts are internally consistent with the pair counts (pairs ≤ structures × organisms)
- Output JSON is written and parses with exactly the four expected count keys

## Limitations

- The curation pipeline is lossy and imperfect: 'the current processing pipeline still removes a lot of correct entries and misses or induces some incorrect ones', so deposited counts are a curated subset, not a ground truth
- Zenodo deposits are versioned (the article lists multiple DOIs such as 10.5281/zenodo.5794106, 10.5281/zenodo.5794597, 10.5281/zenodo.5668854); counts only reproduce for the version matching the reported figures
- The pipeline is heavy with many dependencies (R, Python 3, Java >= 17 per the README), but this skill only requires reading the already-deposited output, not re-running it

## Evidence
