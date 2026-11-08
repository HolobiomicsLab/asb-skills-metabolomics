---
name: recall-true-positive-curve-analysis
description: Use when comparing spectral library search or ranking methods (e.g. MS2Query
  vs MS2Deepscore, cosine, modified cosine) on a labeled test set where each query
  has a known exact-match ground truth and you need a recall-vs-true-positive-rate
  benchmark.
when_to_use_negative:
- Input spectra have no ground-truth annotations, so true positives cannot be identified
  — use analogue-search quality metrics instead
- Input is a feature table or preprocessed peak list (xcms FBMN output) rather than
  ranked query-vs-library search results
- You need analogue (non-identical) match quality; this skill evaluates exact-match
  retrieval only
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3383
- http://edamontology.org/topic_0091
tools:
- name: MS2Query
  role: Candidate method being benchmarked; ranks the full library for each query
    with no precursor m/z pre-filtering (version 0.3.2 used in the benchmark)
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Baseline spectral-similarity method; scores every test query against all library
    spectra and ranks the library accordingly
- name: matchms
  role: Computes the cosine (greedy, 0.05 Da fragment mass tolerance) and modified
    cosine (0.25 Da precursor preselection) baseline scores and handles library cleaning
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:100a24b765b303588f2243ef136f4d1d9706c1691efcf72970d05c25c993d7c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - matchms
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3383
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# recall-true-positive-curve-analysis

## Summary

Benchmark library-search methods by sweeping the top-N ranking cutoff and plotting the percentage of true positives (exact matches) as a function of recall, so methods are compared at equal sensitivity rather than at a single arbitrary cutoff.

## When to use

Use when comparing spectral library search or ranking methods (e.g. MS2Query vs MS2Deepscore, cosine, modified cosine) on a labeled test set where each query has a known exact-match ground truth and you need a recall-vs-true-positive-rate benchmark.

## When NOT to use

- Input spectra have no ground-truth annotations, so true positives cannot be identified — use analogue-search quality metrics instead
- Input is a feature table or preprocessed peak list (xcms FBMN output) rather than ranked query-vs-library search results
- You need analogue (non-identical) match quality; this skill evaluates exact-match retrieval only

## Inputs

- Exact matches test set MS2 query spectra (MGF or matchms Spectrum objects, cleaned and mode-split)
- Cleaned GNPS library spectra (e.g. ALL_GNPS.mgf / ALL_GNPS_NO_PROPOGATED.mgf from Zenodo 10.5281/zenodo.6811540)
- Per-query ranked library results from each method (MS2Query, MS2Deepscore, cosine, modified cosine)
- Ground-truth mapping of test queries to their exact-match library spectrum InChIKeys

## Outputs

- benchmark_exact_matches.csv with columns for method, top-N cutoff, recall, and percentage of true positives
- Recall-versus-true-positive-rate curves for all compared methods

## How to apply

Rank the full library for each test query with every method under comparison — MS2Query with no precursor m/z pre-filtering, MS2Deepscore over all library spectra, greedy cosine from matchms with a fragment mass tolerance of 0.05 Da, and modified cosine restricted to library spectra within 0.25 Da precursor mass tolerance. Sweep the top-N ranking cutoff for each method and compute, at each N, recall (fraction of true exact matches found) and the percentage of returned hits that are true positives. Label a hit as a true positive when the library spectrum is the exact match for the query (within the 0.25 Da mass tolerance used for benchmarking). Assemble the recall-versus-true-positive-rate values for all methods into a single table (e.g. benchmark_exact_matches.csv); the better method maintains a higher true-positive percentage across the recall range.

## Related tools

- **MS2Query** (Candidate method being benchmarked; ranks the full library for each query with no precursor m/z pre-filtering (version 0.3.2 used in the benchmark)) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Baseline spectral-similarity method; scores every test query against all library spectra and ranks the library accordingly)
- **matchms** (Computes the cosine (greedy, 0.05 Da fragment mass tolerance) and modified cosine (0.25 Da precursor preselection) baseline scores and handles library cleaning)

## Examples

```
import pandas as pd; pd.DataFrame(rows, columns=['method','top_n','recall','true_positive_pct']).to_csv('benchmark_exact_matches.csv', index=False)
```

## Evaluation signals

- benchmark_exact_matches.csv contains rows for all four methods (MS2Query, MS2Deepscore, cosine, modified cosine) across the swept top-N cutoffs
- At every recall value, the MS2Query true-positive percentage is higher than that of cosine, modified cosine, and MS2Deepscore
- Exact-match labeling respects the 0.25 Da mass tolerance; cosine hits use the 0.05 Da fragment mass tolerance as specified
- Recall values increase monotonically as the top-N cutoff is relaxed, and true-positive percentage decreases as lower-ranked hits are included
- MS2Query results reproduce the published finding that it performs better at finding exact matches than the cosine-based baselines

## Limitations

- It is hard to objectively judge whether MS2Query performs similarly on newly generated data compared to the benchmarking test set
- Modified cosine baseline depends on the 0.25 Da precursor mass preselection, so it cannot be compared outside that mass window
- Manually validating matches is challenging, making it hard to objectively judge result quality beyond the labeled test set

## Evidence

- [discussion] MS2Query performs better at finding exact matches and analogues for positive mode MS2 spectra than cosine score, modified cosine score, and MS2Deepscore: "Comparison with cosine score, modified cosine score, and MS2Deepscore shows that MS2Query performs better both at finding exact matches as well as finding analogues"
- [discussion] Exact match benchmarking limited to mass tolerance of 0.25 Da: "only library spectra were considered within a mass tolerance of 0.25 Da"
- [discussion] Cosine score computed with fragment mass tolerance of 0.05 Da: "the cosine greedy implementation of matchms33 was used with a fragment mass tolerance of 0.05 Da"
- [intro] MS2Query can reliably predict good analogues as well as exact library matches: "MS2Query can reliably predict good analogues as well as exact library matches"
- [readme] README: MS2Query benchmarking claims better performance than cosine standards: [evidence span withheld — anchor status: external_doc]
