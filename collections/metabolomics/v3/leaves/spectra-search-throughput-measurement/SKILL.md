---
name: spectra-search-throughput-measurement
description: Use when comparing the runtime scalability of MS2 library search methods
  (e.g. MS2Query vs matchms modified cosine) on a fixed query-vs-library task, and
  wall-clock throughput in spectra/minute is the metric of interest.
when_to_use_negative:
- The goal is to compare annotation accuracy or match quality (e.g. exact-match or
  analogue-search performance) rather than runtime throughput.
- The library is small or heavily pre-filtered by precursor m/z, so the full-library-vs-preselection
  speed contrast that this benchmark measures cannot manifest.
- Query spectra are already clustered/feature-reduced such that the per-spectrum throughput
  denominator is not comparable to the 5987-spectrum benchmark setup.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: MS2Query
  role: Searched in version 0.3.2 over all test spectra against the full library with
    no precursor m/z preselection, to measure its throughput
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Used for spectrum cleaning (filtering <3 peaks, intensity normalization) and
    for the optimized modified cosine score implementation whose runtime is benchmarked
    with 100 Da precursor m/z preselection
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:3092e851d2009e471bd9f7b66b94d86b651cf376d177cd600cb9cfd8270e085f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Query
  - matchms
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# spectra-search-throughput-measurement

## Summary

Benchmarks wall-clock runtime throughput (spectra per minute) of MS2 mass spectral library search methods by running each tool over an identical query set against a fixed library on the same hardware. Used to quantify the scalability advantage of MS2Query's embedding-based preselection over modified cosine score-based analogue searching.

## When to use

Use when comparing the runtime scalability of MS2 library search methods (e.g. MS2Query vs matchms modified cosine) on a fixed query-vs-library task, and wall-clock throughput in spectra/minute is the metric of interest.

## When NOT to use

- The goal is to compare annotation accuracy or match quality (e.g. exact-match or analogue-search performance) rather than runtime throughput.
- The library is small or heavily pre-filtered by precursor m/z, so the full-library-vs-preselection speed contrast that this benchmark measures cannot manifest.
- Query spectra are already clustered/feature-reduced such that the per-spectrum throughput denominator is not comparable to the 5987-spectrum benchmark setup.

## Inputs

- GNPS library spectra MGF file (ALL_GNPS_NO_PROPOGATED.mgf, 302,514 spectra, downloaded 1 November 2022)
- Positive-mode test query spectra (5987 MS2 spectra, from Zenodo 10.5281/zenodo.6811540)
- matchms-cleaned library and query spectra (>=3 peaks, normalized intensities)

## Outputs

- benchmark_throughput_results.csv with per-method elapsed wall-clock time and throughput in spectra per minute
- Reported throughput values: MS2Query 80 spectra/minute (1 h 14 min) vs modified cosine 10.6 spectra/minute (9 h 24 min)

## How to apply

Obtain the GNPS library spectra (ALL_GNPS_NO_PROPOGATED.mgf, downloaded 1 November 2022) and the positive-mode test query spectra from Zenodo 10.5281/zenodo.6811540. Clean both library and query spectra with matchms, filtering out spectra with fewer than three peaks and normalizing peak intensities. Run MS2Query version 0.3.2 over the 5987 test spectra against the full 302,514-spectrum library WITHOUT any precursor m/z preselection, recording elapsed wall-clock time; then run the matchms modified cosine score implementation over the same queries after first filtering library spectra to a maximum precursor m/z difference of 100 Da, recording its elapsed time. Compute throughput in spectra per minute for each method and write the timings to benchmark_throughput_results.csv. The speed difference is grounded in MS2Query's design: MS2Deepscore similarities against the full library are computed quickly via pre-calculated mass spectral embeddings, the top 2000 library spectra are selected, and only those are re-ranked with a random forest combining 5 features.

## Related tools

- **MS2Query** (Searched in version 0.3.2 over all test spectra against the full library with no precursor m/z preselection, to measure its throughput) — https://github.com/iomega/ms2query
- **matchms** (Used for spectrum cleaning (filtering <3 peaks, intensity normalization) and for the optimized modified cosine score implementation whose runtime is benchmarked with 100 Da precursor m/z preselection)

## Examples

```
ms2query --spectra ./test_spectra --library ./library_folder --ionmode positive
```

## Evaluation signals

- MS2Query run over 5987 test spectra vs the 302,514-spectrum library completes in approximately 1 hour 14 minutes (~80 spectra/minute) on an Intel Core i5-1135G7 laptop with 16 GB RAM.
- The matchms modified cosine run with 100 Da precursor m/z preselection takes on the order of 9 hours 24 minutes (~10.6 spectra/minute) on the same hardware.
- The measured modified cosine runtime is consistent with the reported ~9 h 24 min scale of computation, and MS2Query is substantially faster (order-of-magnitude throughput advantage).
- Both methods were run over the identical cleaned query set and library, so throughput figures are computed on a like-for-like workload.
- Throughput values are written to benchmark_throughput_results.csv as spectra per minute for both methods.

## Limitations

- Timings are hardware-dependent (Intel Core i5-1135G7, 16 GB RAM in this benchmark); throughput numbers do not transfer to other machines.
- The benchmark used MS2Query version 0.3.2; results are specific to that version and to the 1 November 2022 GNPS library snapshot.
- The comparison conditions differ by design (no precursor m/z preselection for MS2Query vs 100 Da preselection for modified cosine), so the runtime comparison is not a controlled ablation of a single factor.

## Evidence

- [intro] matchms modified cosine with preselection takes ~9.4 hours: "a preselection on a maximum precursor m/z difference of 100 Da took 9 hours and 24 minutes"
- [intro] matchms modified cosine implementation used: "For doing the modified cosine score calculations the implementation of matchms33 was used, which is optimized for performance."
- [intro] MS2Query version pinned for the benchmark: "using version 0.3.2 of MS2Query"
- [discussion] Embedding pre-calculation is the source of the speed advantage: "The biggest speed improvement was achieved by pre-calculating mass spectral embeddings for Spec2Vec and MS2Deepscore."
- [readme] Pre-computed embeddings enable fast full-library comparison without precursor preselection: [evidence span withheld — anchor status: external_doc]
