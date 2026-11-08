---
name: mass-spectral-library-retrieval
description: Use when querying mass spectra against a large spectral library (e.g.
  MoNA-hosted MassBank, GNPS, HMDB, ReSpect, FiehnLib entries) and you need to shrink
  the candidate set before running a full NIST cosine similarity search.
when_to_use_negative:
- The reference library is proprietary NIST-only and not openly deposited on MoNA
  (NIST entries cannot be released publicly, so the prefilter index cannot be built
  from public data)
- The task requires exact full-spectrum identification with maximal recall rather
  than search-space prefiltering, since the second block sacrifices ~13% recall at
  score 700
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Java reference implementation
  role: Generate SPLASH identifiers and extract second-block values via SplashFactory/SplashUtil
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Python
  role: Score query-candidate pairs with NIST cosine similarity and compute search-space
    reduction / recall metrics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:0de5918656f9c7cee2d9645b63535da6a9d7c6cc61523c287ea7f2ceebb5535c
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java
  - Java reference implementation
  - Python
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mass-spectral-library-retrieval

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Uses the SPLASH second summary block as a database-independent prefilter to reduce the mass spectral search space before full-spectrum similarity scoring, trading a small recall loss for large candidate-set reductions.

## When to use

Use when querying mass spectra against a large spectral library (e.g. MoNA-hosted MassBank, GNPS, HMDB, ReSpect, FiehnLib entries) and you need to shrink the candidate set before running a full NIST cosine similarity search.

## When NOT to use

- The reference library is proprietary NIST-only and not openly deposited on MoNA (NIST entries cannot be released publicly, so the prefilter index cannot be built from public data)
- The task requires exact full-spectrum identification with maximal recall rather than search-space prefiltering, since the second block sacrifices ~13% recall at score 700

## Inputs

- Mass spectral library in single-line 'ion:intensity ion:intensity ...' string representation (e.g. from MoNA: MassBank, GNPS, HMDB, ReSpect, FiehnLib spectra)
- Query spectra set (e.g. a 19,435-spectrum query subset) with peak-intensity pairs

## Outputs

- SPLASH identifiers (e.g. splash10-0002-0900000000-b112e4e059e1ecf98c5f) and retained second-block values for all library spectra
- Per-query candidate lists restricted to shared second-block values
- Table of per-query metrics (candidate count, search-space reduction, recall at NIST cosine score 700), e.g. splash_second_block_evaluation.csv

## How to apply

Generate SPLASH identifiers for every library spectrum using the Java reference implementation (SplashFactory/SplashUtil), retaining the second summary block of each SPLASH. For each query spectrum, restrict the reference set to spectra sharing the same second-block value to form the candidate list, then score every query-candidate pair with the NIST cosine similarity on the peak-intensity vectors and rank candidates. The selected second block, chosen from 136 potential block formats, reduced the search space by 94% or above (36,107 spectra or fewer out of 563,902) in all cases while returning 87% of spectra within a NIST cosine similarity score of 700. Record per-query candidate counts, search-space reduction, and recall-at-700 to judge correctness.

## Related tools

- **Java reference implementation** (Generate SPLASH identifiers and extract second-block values via SplashFactory/SplashUtil) — https://github.com/berlinguyinca/spectra-hash
- **Python** (Score query-candidate pairs with NIST cosine similarity and compute search-space reduction / recall metrics)

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS);
```

## Evaluation signals

- Search-space reduction is ≥94% for every query (≤36,107 candidate spectra remaining from a 563,902-spectrum reference set)
- Recall: ~87% of queries return their true match within a NIST cosine similarity score of 700
- Generated SPLASHes match the Java reference implementation output when checked with the validation tool (java -jar validation/target/validation-1.8.jar -c -s 2 -t ms)
- Identical second-block grouping reproduces known duplicate spectra (e.g. caffeine as PR100026/MassBank, 5464/HMDB, CCMSLIB00000222858/GNPS)
- Per-query metrics CSV contains candidate counts and reduction/recall columns with no missing values

## Limitations

- 87% recall at score 700 means roughly 13% of true matches are excluded by the second-block prefilter
- NIST library spectra are excluded from the public MoNA deposit, so the public 563,902-spectrum dataset does not include NIST entries
- SPLASH hash collisions in the fourth block were only observed for single-ion spectra of the same mass; collision probability is ~10^-31 for a database of 10^9 spectra

## Evidence
