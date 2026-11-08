---
name: chemical-occurrence-record-deduplication
description: Use when a deposited, curated natural-products occurrence table (e.g.
  LOTUS 3_curated/table.tsv.gz from Zenodo DOI 10.5281/zenodo.5607264) must be tallied
  for unique structure-organism pairs to validate a reported figure such as 750,000+
  pairs.
when_to_use_negative:
- Input is the raw initial table (2.5M+ pre-validation pairs) rather than the curated
  3_curated table — deduplication counts on unharmonized data will not reproduce the
  reported figures
- The goal is applying curation/filtering rules (manual validation of 420 pairs, filter_dirty.R,
  sanitization via MolVS) rather than counting the already-curated output
- Counts are needed from the Wikidata mirror via SPARQL queries rather than from the
  deposited flat TSV table
tools:
- name: pandas
  role: Loads the gzip-compressed TSV (sep='\t', compression='gzip') and performs
    duplicate-dropping to count unique structure-organism pairs
- name: lotus-processor
  role: Source pipeline whose Zenodo deposit (DOI 10.5281/zenodo.5607264) provides
    the curated table.tsv.gz input being counted
  repo: https://github.com/lotusnprod/lotus-processor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:f220fcdd3b8784a41f6d1540f4aa4481ff9f2e417a0ca5727e095180552bfaaf
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-processor
  tools:
  - lotus-processor
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# chemical-occurrence-record-deduplication

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Counts unique referenced structure-organism pairs in a curated LOTUS deposit table at full stereochemical (3D) and flat (2D) levels, cross-checking structure and organism tallies to confirm the reported dissemination total of 750,000+ pairs (~30% of 2.5M+ initial pairs).

## When to use

Use when a deposited, curated natural-products occurrence table (e.g. LOTUS 3_curated/table.tsv.gz from Zenodo DOI 10.5281/zenodo.5607264) must be tallied for unique structure-organism pairs to validate a reported figure such as 750,000+ pairs.

## When NOT to use

- Input is the raw initial table (2.5M+ pre-validation pairs) rather than the curated 3_curated table — deduplication counts on unharmonized data will not reproduce the reported figures
- The goal is applying curation/filtering rules (manual validation of 420 pairs, filter_dirty.R, sanitization via MolVS) rather than counting the already-curated output
- Counts are needed from the Wikidata mirror via SPARQL queries rather than from the deposited flat TSV table

## Inputs

- Gzip-compressed TSV file interim/tables/3_curated/table.tsv.gz from the lotus-processor Zenodo deposit (DOI 10.5281/zenodo.5607264)
- Structure-organism-reference occurrence records with curated chemical structure, biological organism and reference columns

## Outputs

- structure_organism_pair_counts.json containing unique referenced pair counts at 3D and 2D levels plus unique structure and organism counts
- Validated pair totals expected to match 588694 (3D) and 484174 (2D)

## How to apply

Download the lotus-processor Zenodo deposit (DOI 10.5281/zenodo.5607264) and locate interim/tables/3_curated/table.tsv.gz — the table containing entries with a curated chemical structure, biological organism and reference. Load the gzip-compressed TSV into a pandas DataFrame with pd.read_csv(path, sep='\t', compression='gzip'), then count unique referenced structure-organism pairs by dropping duplicate rows, computing the count once at the full stereochemical (3D) level and once at the flat (2D) level. As a cross-check, also tabulate the number of unique curated structures and unique contributing organisms, and write all counts (3D and 2D pairs, structures, organisms) to structure_organism_pair_counts.json. Rationale: the article reports that manual and automatic validation retained 750,000+ pairs from 2.5M+ initial pairs (~30%), so the computed totals must reproduce the expected 3D and 2D figures to confirm the deposit matches the published dissemination.

## Related tools

- **pandas** (Loads the gzip-compressed TSV (sep='\t', compression='gzip') and performs duplicate-dropping to count unique structure-organism pairs)
- **lotus-processor** (Source pipeline whose Zenodo deposit (DOI 10.5281/zenodo.5607264) provides the curated table.tsv.gz input being counted) — https://github.com/lotusnprod/lotus-processor

## Examples

```
import pandas as pd; df = pd.read_csv('interim/tables/3_curated/table.tsv.gz', sep='\t', compression='gzip'); pairs = df.drop_duplicates()
```

## Evaluation signals

- Computed unique referenced structure-organism pair count at the 3D stereochemical level equals the reported 588694
- Computed unique pair count at the 2D flat level equals the reported 484174
- Overall retained total is consistent with the reported 750,000+ pairs, i.e. approximately 30% of the 2.5M+ initial pairs
- Cross-check tabulation of unique curated structures and unique contributing organisms is produced alongside the pair counts
- Output JSON structure_organism_pair_counts.json contains both pair-level counts plus structure and organism counts

## Limitations

- The underlying LOTUS processing pipeline still removes many correct entries and misses or induces some incorrect ones, so deduplication counts reflect the curation state, not ground truth
- 3D and 2D deduplication levels give different totals (588694 vs 484174); reporting only one level will not match all published figures
- The pipeline is heavy with many dependencies and is convoluted, so reproducing the deposited table from source (rather than using the Zenodo deposit) is non-trivial

## Evidence
