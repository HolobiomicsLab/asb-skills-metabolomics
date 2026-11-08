---
name: precursor-mass-preselection-filtering
description: Use when running modified cosine (or similar score-based) analogue searching
  against a large library (e.g., 302,514 GNPS spectra) and you need to bound pairwise
  score computations by precursor m/z proximity, or when benchmarking the runtime
  cost of such preselection.
when_to_use_negative:
- When running MS2Query-style embedding-based search (MS2Deepscore + random forest
  re-ranking), which deliberately performs no precursor m/z preselection
- When the goal is exact library matching with a tight mass tolerance (e.g., 0.25
  Da), which is a different filtering regime
- When candidate analogues are expected to lie outside the chosen precursor m/z window,
  since preselection will discard them before scoring
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: matchms
  role: Provides the modified cosine score implementation used with the 100 Da precursor
    m/z preselection filter; also used to clean spectra before filtering
- name: MS2Query 0.3.2
  role: Baseline comparator that runs full-library analogue search without precursor
    m/z preselection, used to quantify the preselection's runtime cost
  repo: https://github.com/iomega/ms2query
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:186d9873d5ce41200284fa7351066ad6c3c931c1d44535939524286a40787ba4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Query
  - MS2Query 0.3.2
  - matchms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# precursor-mass-preselection-filtering

## Summary

A library-spectra preselection filter that restricts candidate matches to those within a maximum precursor m/z difference (e.g., 100 Da) before spectral similarity score computation, as used for the matchms modified cosine analogue search benchmarked against MS2Query.

## When to use

Use when running modified cosine (or similar score-based) analogue searching against a large library (e.g., 302,514 GNPS spectra) and you need to bound pairwise score computations by precursor m/z proximity, or when benchmarking the runtime cost of such preselection.

## When NOT to use

- When running MS2Query-style embedding-based search (MS2Deepscore + random forest re-ranking), which deliberately performs no precursor m/z preselection
- When the goal is exact library matching with a tight mass tolerance (e.g., 0.25 Da), which is a different filtering regime
- When candidate analogues are expected to lie outside the chosen precursor m/z window, since preselection will discard them before scoring

## Inputs

- Positive-mode test query MS2 spectra (MGF, e.g., from Zenodo 10.5281/zenodo.6811540)
- GNPS library spectra (ALL_GNPS_NO_PROPOGATED.mgf, downloaded 1 November 2022)
- Precursor m/z difference threshold in Da (e.g., 100 Da)

## Outputs

- Per-query set of library spectra within the precursor m/z tolerance for score computation
- Modified cosine analogue search results with elapsed wall-clock timing
- Throughput values in spectra per minute (e.g., benchmark_throughput_results.csv)

## How to apply

After cleaning the library and query spectra with matchms (filtering out spectra with fewer than three peaks and normalizing peak intensities), filter the library spectra on a maximum precursor m/z difference (100 Da in the MS2Query benchmark) relative to each query spectrum, then compute modified cosine scores only on the retained candidates. This trades analogue-discovery range for compute time: the article reports the preselected modified cosine search of 5987 test spectra against the 302,514-spectrum ALL_GNPS_NO_PROPOGATED.mgf library took 9 hours and 24 minutes (10.6 spectra per minute) on an Intel Core i5-1135G7 with 16 GB RAM, versus 1 hour 14 minutes (80 spectra per minute) for MS2Query 0.3.2 running without any precursor m/z preselection. Record elapsed wall-clock time per method and compute throughput in spectra per minute for comparison.

## Related tools

- **matchms** (Provides the modified cosine score implementation used with the 100 Da precursor m/z preselection filter; also used to clean spectra before filtering)
- **MS2Query 0.3.2** (Baseline comparator that runs full-library analogue search without precursor m/z preselection, used to quantify the preselection's runtime cost) — https://github.com/iomega/ms2query

## Evaluation signals

- Every candidate library spectrum retained per query satisfies the maximum precursor m/z difference threshold (100 Da in the benchmark)
- Measured modified cosine run time is consistent with the reported ~9 hours and 24 minutes scale for 5987 query spectra against the 302,514-spectrum library
- Computed throughput (~10.6 spectra per minute with 100 Da preselection vs ~80 spectra per minute for MS2Query without preselection) matches the reported order-of-magnitude gap
- Spectra with fewer than three peaks are excluded and peak intensities are normalized before filtering/scoring

## Limitations

- Preselection restricts analogues to a fixed precursor m/z window, unlike MS2Query which performs no precursor m/z preselection
- The filtering strategy is substantially slower: 9 h 24 min versus 1 h 14 min for the same 5987-spectrum query against the 302,514-spectrum library
- Benchmarked on a specific hardware setup (Intel Core i5-1135G7, 16 GB RAM) using a GNPS library snapshot dated 1 November 2022

## Evidence

- [intro] Preselect library spectra by maximum precursor m/z difference: "a preselection on a maximum precursor m/z difference of 100 Da took 9 hours and 24 minutes"
- [discussion] Analogue search with modified cosine filtered to 100 Da: "library spectra were first filtered on a maximum precursor m/z difference of 100 Da"
- [readme] MS2Query avoids precursor m/z preselection: [evidence span withheld — anchor status: external_doc]
- [readme] Pre-calculated embeddings enable full-library comparison: [evidence span withheld — anchor status: external_doc]
