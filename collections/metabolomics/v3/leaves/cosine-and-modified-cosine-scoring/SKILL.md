---
name: cosine-and-modified-cosine-scoring
description: Use when MS2 query spectra must be compared against library spectra with
  established peak-matching similarity scores (cosine or modified cosine) to establish
  a baseline for exact-match or analogue search benchmarks.
when_to_use_negative:
- When the goal is analogue search beyond a 0.25 Da precursor mass window — modified
  cosine preselection excludes analogues with larger precursor differences, so a learned
  similarity (MS2Deepscore/MS2Query) is needed instead
- When the query spectra have not been peak-picked or clustered and contain many redundant
  MS2 spectra per feature — MS2Query-based skills assume preprocessed input
edam_operation: http://edamontology.org/operation_3797
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Provides the greedy cosine implementation (fragment mass tolerance 0.05 Da)
    and the modified cosine score used for benchmark comparisons
- name: MS2Query
  role: The method being benchmarked against cosine and modified cosine baselines;
    version 0.3.2 used in the study
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Additional learned-similarity baseline compared against cosine/modified cosine
    rankings
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:794d910e5cca5d1b5ba301cbf74a771932db4f6238c109af61d5b1e85d92bf5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - matchms
  edam_operation: http://edamontology.org/operation_3797
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cosine-and-modified-cosine-scoring

## Summary

Baseline spectral similarity scoring for MS2 library matching using the greedy cosine score (fragment mass tolerance 0.05 Da) and the matchms modified cosine score with 0.25 Da precursor m/z preselection, as used to benchmark MS2Query's exact-match and analogue search performance.

## When to use

Use when MS2 query spectra must be compared against library spectra with established peak-matching similarity scores (cosine or modified cosine) to establish a baseline for exact-match or analogue search benchmarks.

## When NOT to use

- When the goal is analogue search beyond a 0.25 Da precursor mass window — modified cosine preselection excludes analogues with larger precursor differences, so a learned similarity (MS2Deepscore/MS2Query) is needed instead
- When the query spectra have not been peak-picked or clustered and contain many redundant MS2 spectra per feature — MS2Query-based skills assume preprocessed input

## Inputs

- Test query MS2 spectra (e.g. from the exact matches test set in the Zenodo deposit 10.5281/zenodo.6811540)
- Cleaned GNPS library spectra (matchms-cleaned MGF, e.g. ALL_GNPS.mgf or ALL_GNPS_NO_PROPOGATED.mgf)

## Outputs

- Query-by-library similarity score matrix for cosine and modified cosine
- Ranked library spectra per query spectrum
- Recall-versus-true-positive-percentage benchmark table (e.g. benchmark_exact_matches.csv)

## How to apply

Clean the GNPS library and test spectra with matchms (uniform metadata, peak filtering) and load them as matchms Spectrum objects. Compute cosine scores with the greedy matchms implementation using a fragment mass tolerance of 0.05 Da between every test query and all library spectra, ranking the library per query. For modified cosine, preselect only library spectra within a 0.25 Da precursor mass tolerance of each query before scoring, which restricts matches to near-identical precursor masses. For analogue search benchmarks, additionally filter library spectra to a maximum precursor m/z difference of 100 Da. Sweep the top-N ranking cutoff and record recall versus percentage of true positives (exact matches) to compare against MS2Query and MS2Deepscore rankings.

## Related tools

- **matchms** (Provides the greedy cosine implementation (fragment mass tolerance 0.05 Da) and the modified cosine score used for benchmark comparisons)
- **MS2Query** (The method being benchmarked against cosine and modified cosine baselines; version 0.3.2 used in the study) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Additional learned-similarity baseline compared against cosine/modified cosine rankings)

## Examples

```
from matchms import calculate_scores; scores = calculate_scores(queries, library, ModifiedCosineGreedyScore(tolerance=0.25))
```

## Evaluation signals

- Fragment mass tolerance for cosine is set to 0.05 Da, matching the article's benchmark configuration
- Modified cosine is restricted to library spectra within 0.25 Da precursor mass tolerance of each query
- In the exact-match benchmark, cosine and modified cosine should show a lower percentage of true positives across recall values than MS2Query without precursor m/z pre-filtering
- Recall versus true-positive-percentage values are swept over top-N ranking cutoffs and assembled into a benchmark CSV

## Limitations

- Modified cosine's precursor mass preselection (0.25 Da; 100 Da for analogue search) makes scoring slow on large libraries — a 100 Da preselection took 9 hours and 24 minutes
- Cosine and modified cosine rely on peak overlap and are weaker proxies for chemical similarity of non-identical molecules than embedding-based scores
- Software versions for matchms are not stated in the article

## Evidence

- [discussion] the cosine greedy implementation of matchms33 was used with a fragment mass tolerance of 0.05 Da: "the cosine greedy implementation of matchms33 was used with a fragment mass tolerance of 0.05 Da"
- [discussion] exact match benchmarking mass tolerance: "only library spectra were considered within a mass tolerance of 0.25 Da"
- [intro] modified cosine via matchms: "For doing the modified cosine score calculations the implementation of matchms33 was used, which is optimized for performance."
- [intro] analogue search precursor preselection: "a preselection on a maximum precursor m/z difference of 100 Da took 9 hours and 24 minutes"
- [readme] benchmark outcome vs MS2Query: [evidence span withheld — anchor status: external_doc]
