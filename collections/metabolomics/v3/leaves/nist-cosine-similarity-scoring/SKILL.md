---
name: nist-cosine-similarity-scoring
description: Use when a reduced candidate set of mass spectra (e.g. grouped by SPLASH
  second-block value) must be scored and ranked against a query spectrum to quantify
  search-space reduction versus similarity-based recall.
when_to_use_negative:
- Spectra are not represented as peak-intensity vectors suitable for cosine scoring
  (e.g. raw profile-mode data not yet peak-picked)
- The goal is identity assignment via the full SPLASH hash block rather than similarity-based
  recall over a prefiltered candidate set
- The reference library is the NIST 14 library itself, which cannot be released publicly
  and is excluded from the open MoNA deposit
edam_operation: http://edamontology.org/operation_0400
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Java
  role: Runs the SPLASH reference implementation (SplashFactory/SplashUtil) to generate
    the second-block values used to form candidate lists prior to similarity scoring
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Python
  role: Reference implementation language listed for SPLASH; used alongside Java for
    scoring and recording per-query metrics
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:f1df4af7a828707a63e907a5ad07cfa2e8b0b866f6ab5be5742103df7b792320
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java
  - Python
  edam_operation: http://edamontology.org/operation_0400
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# nist-cosine-similarity-scoring

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Scores query-reference mass spectrum pairs with the NIST cosine similarity computed on peak-intensity vectors, then ranks candidates and evaluates recall at a fixed score threshold. In the SPLASH second-block evaluation it measures whether spectra sharing the same second summary block recover the true match (87% recall at a score of 700).

## When to use

Use when a reduced candidate set of mass spectra (e.g. grouped by SPLASH second-block value) must be scored and ranked against a query spectrum to quantify search-space reduction versus similarity-based recall.

## When NOT to use

- Spectra are not represented as peak-intensity vectors suitable for cosine scoring (e.g. raw profile-mode data not yet peak-picked)
- The goal is identity assignment via the full SPLASH hash block rather than similarity-based recall over a prefiltered candidate set
- The reference library is the NIST 14 library itself, which cannot be released publicly and is excluded from the open MoNA deposit

## Inputs

- Query spectra subset (19,435 spectra) from the 563,902-spectrum MoNA deposit (MassBank, GNPS, HMDB, ReSpect, FiehnLib; NIST entries excluded)
- SPLASH identifiers with second summary block for all spectra, generated with the Java reference implementation (SplashFactory/SplashUtil)
- Peak-intensity vectors for each query-candidate pair

## Outputs

- Per-query candidate counts and search-space reduction values
- Recall fraction at NIST cosine similarity score 700
- splash_second_block_evaluation.csv with per-query metrics

## How to apply

For each of the 19,435 query spectra, restrict the 563,902-spectrum reference set (MassBank, GNPS, HMDB, ReSpect, FiehnLib via MoNA; NIST entries excluded from the public deposit) to spectra sharing the same SPLASH second-block value. Score every query-candidate pair with the NIST cosine similarity computed on the peak-intensity vectors of the spectra, and rank the candidates by score. Record the number of remaining candidates per query to compute the search-space reduction (94% or above, i.e. 36,107 spectra or fewer remaining, was the reported criterion), and compute the fraction of queries whose true match is returned within a similarity score of 700 (reported: 87%). Write per-query metrics to splash_second_block_evaluation.csv.

## Related tools

- **Java** (Runs the SPLASH reference implementation (SplashFactory/SplashUtil) to generate the second-block values used to form candidate lists prior to similarity scoring) — https://github.com/berlinguyinca/spectra-hash
- **Python** (Reference implementation language listed for SPLASH; used alongside Java for scoring and recording per-query metrics) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
java -jar validation/target/validation-1.8.jar -c -s 2 -t ms base-dataset/spectra/notsplashed/test-set-v1.csv base-dataset/spectra/test-set-with-splash-v1.csv
```

## Evaluation signals

- Search-space reduction is 94% or above in all cases, i.e. 36,107 candidate spectra or fewer remain per query
- 87% of all queries return their true match within a NIST cosine similarity score of 700
- Per-query metrics are written to splash_second_block_evaluation.csv with candidate count, reduction, and recall-at-700 columns
- Query and reference spectra are restricted to identical second-block values before scoring, so candidate list sizes must not exceed the full reference set

## Limitations

- 87% recall means 13% of true matches are not returned within a similarity score of 700 even after second-block prefiltering
- NIST entries cannot be released publicly, so the openly deposited 563,902-spectrum dataset excludes NIST spectra
- The second block was selected from 136 candidate block formats; other SPLASH blocks may give different reduction/recall trade-offs

## Evidence

- [other] Score every query-candidate pair with the NIST cosine similarity on the peak-intensity vectors and rank the candidates.: [evidence span withheld — anchor status: not_found]
