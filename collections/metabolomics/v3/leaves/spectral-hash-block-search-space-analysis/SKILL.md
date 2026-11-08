---
name: spectral-hash-block-search-space-analysis
description: Use when a spectral library or query set must be partitioned into small
  candidate groups before similarity searching, or when testing whether a SPLASH block
  format reduces search space without losing true matches.
when_to_use_negative:
- When the goal is unique spectrum identification by the fourth SHA256-derived hash
  block rather than candidate-set prefiltering by the second block
- When input is a chemical-structure identifier task (e.g. InChIKey matching) rather
  than mass spectral similarity search
- When the reference library cannot be represented as peak-intensity vectors compatible
  with NIST cosine scoring
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Java SPLASH reference implementation (SplashFactory/SplashUtil)
  role: Generates SPLASH identifiers from spectra; the second summary block is retained
    to bucket spectra into candidate groups
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Python
  role: Used with Java to score query-candidate pairs with NIST cosine similarity
    and write per-query evaluation metrics to CSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:29d3625e6106f6b93956a4eecf8c782a68fffd28b94eff3bcd9f0fed47546bc3
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java
  - Java SPLASH reference implementation (SplashFactory/SplashUtil)
  - Python
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# spectral-hash-block-search-space-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Evaluates how a SPLASH identifier block (the second summary block) acts as a mass spectral search prefilter: it buckets spectra sharing the same block value and measures search-space reduction and NIST-cosine recall against the full reference set.

## When to use

Use when a spectral library or query set must be partitioned into small candidate groups before similarity searching, or when testing whether a SPLASH block format reduces search space without losing true matches.

## When NOT to use

- When the goal is unique spectrum identification by the fourth SHA256-derived hash block rather than candidate-set prefiltering by the second block
- When input is a chemical-structure identifier task (e.g. InChIKey matching) rather than mass spectral similarity search
- When the reference library cannot be represented as peak-intensity vectors compatible with NIST cosine scoring

## Inputs

- Mass spectral library dataset of 563,902 spectra from MassBank, GNPS, HMDB, ReSpect, FiehnLib and NIST 14 (public portion deposited on MoNA)
- Query subset of 19,435 spectra with single-line 'ion:intensity ion:intensity ...' string representations
- SPLASH identifiers with second summary block retained for every spectrum

## Outputs

- splash_second_block_evaluation.csv with per-query candidate counts, search-space reduction, and recall at similarity score 700
- Aggregate search-space reduction (percentage and remaining candidate count, e.g. 36,107 spectra or fewer)
- Recall fraction of queries whose true match is within NIST cosine similarity score 700

## How to apply

Download the openly deposited 563,902-spectrum dataset from MassBank, GNPS, HMDB, ReSpect and FiehnLib via MoNA (NIST entries are excluded). Generate SPLASH identifiers for all spectra with the Java reference implementation (SplashFactory or SplashUtil), retaining only the second summary block of each identifier. For each of the 19,435 query spectra, restrict the reference set to spectra sharing the same second-block value to form the candidate list. Score every query-candidate pair with the NIST cosine similarity on peak-intensity vectors and rank candidates. Record remaining candidates per query, compute search-space reduction, and compute the fraction of queries whose true match is returned within a NIST similarity score of 700; write per-query metrics to splash_second_block_evaluation.csv. Success criteria are >=94% reduction (<=36,107 remaining candidates) in all cases and 87% recall at score 700.

## Related tools

- **Java SPLASH reference implementation (SplashFactory/SplashUtil)** (Generates SPLASH identifiers from spectra; the second summary block is retained to bucket spectra into candidate groups) — https://github.com/berlinguyinca/spectra-hash
- **Python** (Used with Java to score query-candidate pairs with NIST cosine similarity and write per-query evaluation metrics to CSV)

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS);
```

## Evaluation signals

- Search-space reduction is at least 94% (36,107 spectra or fewer remaining) for every query
- 87% of all spectra are returned within a NIST similarity score of 700
- Per-query metrics CSV exists with one row per query in the 19,435-spectrum query subset
- Each candidate list contains only spectra whose second SPLASH block equals the query's second block
- Generated SPLASH identifiers validate against the reference implementation via the validation tool

## Limitations

- NIST library entries cannot be released publicly, so the open deposit via MoNA excludes them from the reproduced 563,902-spectrum dataset
- The second block was selected from 136 candidate block formats as the best short, web search-compatible prefilter; other blocks may perform differently
- Identical SPLASHes in collision testing on 53.25 million spectra arose only from single-ion spectra of the same mass, but theoretical collision probability is only approximately 10^-31 for a database of 10^9 spectra

## Evidence
