---
name: mass-spectral-library-search-benchmarking
description: Use when comparing the runtime scalability of MS2 spectral library search
  methods (e.g., ML-based analogue search vs score-based search with precursor preselection)
  on a fixed query set and library.
when_to_use_negative:
- When the goal is to benchmark annotation accuracy or analogue-search quality rather
  than runtime throughput
- When query files contain many MS2 spectra per feature and no prior clustering/feature
  selection has been done — MS2Query does not do peak picking or clustering, so unprocessed
  files inflate runtimes and make timings non-comparable
- When the target library or query mode differs (e.g., negative mode) without downloading
  the corresponding pretrained model/library files
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2Query
  role: Runs the full-library analogue/exact-match search over the 5987 test spectra
    with no precursor m/z preselection; timed to measure throughput (v0.3.2)
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Provides the modified cosine score implementation (run with 100 Da precursor
    m/z preselection for timing comparison) and spectrum cleaning filters (fewer than
    three peaks removed, intensity normalization)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:629ec79928a2d54c689f0a9adb26031b2075c863c5af33519d14d23e3d9e7e44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Query
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mass-spectral-library-search-benchmarking

## Summary

Benchmark the wall-clock throughput of MS2-based library/analogue search tools (MS2Query vs matchms modified cosine) by timing full-library matching runs over a fixed query set and reporting spectra-per-minute.

## When to use

Use when comparing the runtime scalability of MS2 spectral library search methods (e.g., ML-based analogue search vs score-based search with precursor preselection) on a fixed query set and library.

## When NOT to use

- When the goal is to benchmark annotation accuracy or analogue-search quality rather than runtime throughput
- When query files contain many MS2 spectra per feature and no prior clustering/feature selection has been done — MS2Query does not do peak picking or clustering, so unprocessed files inflate runtimes and make timings non-comparable
- When the target library or query mode differs (e.g., negative mode) without downloading the corresponding pretrained model/library files

## Inputs

- GNPS library spectra MGF file (ALL_GNPS_NO_PROPOGATED.mgf, downloaded 1 November 2022)
- Positive-mode test query spectra (5987 MS2 spectra, Zenodo 10.5281/zenodo.6811540)
- MS2Query pretrained model and library folder (sqlite library, models, precalculated embeddings)

## Outputs

- benchmark_throughput_results.csv with elapsed wall-clock time and throughput (spectra/minute) per method
- Raw search results per method (MS2Query csv results; matchms modified cosine scores)

## How to apply

Obtain the GNPS library spectra (ALL_GNPS_NO_PROPOGATED.mgf) and the positive-mode test query spectra from Zenodo 10.5281/zenodo.6811540. Clean both library and query spectra with matchms, filtering out spectra with fewer than three peaks and normalizing peak intensities. Run MS2Query 0.3.2 over the 5987 test spectra against the full 302,514-spectrum library with no precursor m/z preselection, recording elapsed wall-clock time; run the matchms modified cosine implementation on the same queries after prefiltering library spectra on a maximum precursor m/z difference of 100 Da. Compute spectra-per-minute throughput for each method (e.g., ~80/min for MS2Query vs ~10.6/min for modified cosine, both on an Intel Core i5-1135G7 laptop with 16 GB RAM) and write timings to benchmark_throughput_results.csv. MS2Query's speed advantage stems from pre-calculated mass spectral embeddings for Spec2Vec and MS2Deepscore plus top-2000 MS2Deepscore preselection before random forest re-ranking.

## Related tools

- **MS2Query** (Runs the full-library analogue/exact-match search over the 5987 test spectra with no precursor m/z preselection; timed to measure throughput (v0.3.2)) — https://github.com/iomega/ms2query
- **matchms** (Provides the modified cosine score implementation (run with 100 Da precursor m/z preselection for timing comparison) and spectrum cleaning filters (fewer than three peaks removed, intensity normalization))

## Examples

```
ms2query --spectra .\location_of_spectra --library .\library_folder --ionmode positive
```

## Evaluation signals

- MS2Query throughput should land near the reported ~80 spectra per minute (5987 spectra in ~1 h 14 min on a comparable laptop)
- Modified cosine throughput should land near ~10.6 spectra per minute (5987 spectra in ~9 h 24 min) with the 100 Da precursor m/z preselection filter applied
- Confirm the MS2Query run shows the order-of-magnitude throughput advantage over modified cosine searching
- Verify both runs used identically cleaned spectra: fewer than three peaks filtered out and intensities normalized
- Check benchmark_throughput_results.csv contains timing and throughput values for both methods on the same query/library setup

## Limitations

- Timings are hardware-dependent (benchmark used an Intel Core i5-1135G7 laptop with 16 GB RAM) and should not be compared across different machines without normalization
- MS2Query does no peak picking or clustering of similar MS2 spectra; inputs with many MS2 spectra per feature must be preprocessed (e.g., via MZMine FBMN output) or runtime will be inflated
- Results are pinned to specific software versions (MS2Query 0.3.2) and library snapshot dates (GNPS download 1 November 2022); re-running with other versions or updated libraries changes both speed and accuracy

## Evidence

- [intro] For doing the modified cosine score calculations the implementation of matchms33 was used, which is optimized for performance.: "For doing the modified cosine score calculations the implementation of matchms33 was used, which is optimized for performance."
- [intro] a preselection on a maximum precursor m/z difference of 100 Da took 9 hours and 24 minutes: "a preselection on a maximum precursor m/z difference of 100 Da took 9 hours and 24 minutes"
- [discussion] The biggest speed improvement was achieved by pre-calculating mass spectral embeddings for Spec2Vec and MS2Deepscore.: "The biggest speed improvement was achieved by pre-calculating mass spectral embeddings for Spec2Vec and MS2Deepscore."
- [readme] In our benchmarking we show that MS2Query performs better compared to current standards in the field like Cosine Score and the Modified Cosine score.: [evidence span withheld — anchor status: external_doc]
- [readme] MS2Query does not do any peak picking or clustering of similar MS2 spectra.: [evidence span withheld — anchor status: external_doc]
