---
name: modified-cosine-score-computation
description: Use when benchmarking library-search throughput or computing modified
  cosine analogue matches between MS2 query spectra and a large GNPS library (~302,514
  spectra) with precursor m/z preselection.
when_to_use_negative:
- 'When search speed at library scale is the priority: the 100 Da precursor preselection
  plus modified cosine scoring took 9 h 24 min for 5987 spectra versus 1 h 14 min
  for MS2Query on the same laptop hardware'
- When analogues beyond a 100 Da precursor m/z difference must be found, since this
  method's analogue search is defined by that preselection, whereas MS2Query performs
  no precursor m/z preselection
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Provides the modified cosine score implementation (optimized for performance)
    used to score query spectra against the preselected GNPS library and to clean
    spectra (>=3 peaks, intensity normalization)
- name: MS2Query
  role: Embedding-based comparator whose throughput (80 spectra/minute) and accuracy
    are contrasted against the modified cosine baseline
  repo: https://github.com/iomega/ms2query
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:9f40e32f926b6a69ac0e31171848ce5292f056f458f8f198628e57625f194e6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Query
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# modified-cosine-score-computation

## Summary

Computes modified cosine scores between positive-mode MS2 query spectra and a GNPS spectral library using the matchms implementation, with a 100 Da precursor m/z preselection, serving as the classical analogue-search baseline in the MS2Query benchmark.

## When to use

Use when benchmarking library-search throughput or computing modified cosine analogue matches between MS2 query spectra and a large GNPS library (~302,514 spectra) with precursor m/z preselection.

## When NOT to use

- When search speed at library scale is the priority: the 100 Da precursor preselection plus modified cosine scoring took 9 h 24 min for 5987 spectra versus 1 h 14 min for MS2Query on the same laptop hardware
- When analogues beyond a 100 Da precursor m/z difference must be found, since this method's analogue search is defined by that preselection, whereas MS2Query performs no precursor m/z preselection

## Inputs

- Positive-mode MS2 test query spectra (5987 spectra, from Zenodo 10.5281/zenodo.6811540)
- GNPS library spectra file ALL_GNPS_NO_PROPOGATED.mgf (302,514 spectra, downloaded 1 November 2022)
- Precursor m/z difference threshold of 100 Da for library preselection

## Outputs

- Modified cosine scores between each query spectrum and preselected library spectra
- benchmark_throughput_results.csv with elapsed run time and throughput in spectra per minute

## How to apply

First clean library and query spectra with matchms, filtering out spectra with fewer than three peaks and normalizing peak intensities. Before scoring, filter the 302,514-spectrum GNPS library (ALL_GNPS_NO_PROPOGATED.mgf) on a maximum precursor m/z difference of 100 Da against each query spectrum — this preselection is what bounds the pairwise comparisons for the analogue search. Then run the matchms modified cosine score implementation over the 5987 positive-mode test spectra against the preselected library entries, recording elapsed wall-clock time. Finally compute throughput in spectra per minute (the reported baseline was 9 hours 24 minutes for 5987 spectra, i.e. 10.6 spectra/minute on an Intel Core i5-1135G7 with 16 GB RAM) and write timing values to benchmark_throughput_results.csv. The rationale is that modified cosine is the standard fragment-peak-matching analogue-search baseline against which embedding-based methods like MS2Query (80 spectra/minute, no precursor preselection) are compared.

## Related tools

- **matchms** (Provides the modified cosine score implementation (optimized for performance) used to score query spectra against the preselected GNPS library and to clean spectra (>=3 peaks, intensity normalization))
- **MS2Query** (Embedding-based comparator whose throughput (80 spectra/minute) and accuracy are contrasted against the modified cosine baseline) — https://github.com/iomega/ms2query

## Evaluation signals

- Measured run time for 5987 test spectra against the 302,514-spectrum library should be on the order of ~9 hours (reported: 9 h 24 min, ~10.6 spectra/minute)
- Library spectra must be filtered on a maximum precursor m/z difference of 100 Da before score computation
- Spectra with fewer than three peaks must be removed and peak intensities normalized during matchms cleaning
- Throughput values (spectra per minute) for both modified cosine and MS2Query runs must be recorded in benchmark_throughput_results.csv
- The MS2Query comparison run must complete substantially faster (~80 spectra/minute, no precursor preselection), confirming the order-of-magnitude throughput gap

## Limitations

- Current analogue search implementations, including modified cosine with preselection, are 'not yet very reliable and relatively slow' per the article
- The 100 Da precursor m/z preselection caps the chemical neighbourhood explored, unlike MS2Query which skips precursor preselection entirely
- Benchmark runtime depends on hardware; reported figures assume a normal laptop (Intel Core i5-1135G7, 16 GB RAM)

## Evidence

- [intro] matchms modified cosine implementation: "For doing the modified cosine score calculations the implementation of matchms33 was used, which is optimized for performance."
- [intro] 100 Da preselection runtime: "a preselection on a maximum precursor m/z difference of 100 Da took 9 hours and 24 minutes"
- [discussion] library filtering before analogue search: "library spectra were first filtered on a maximum precursor m/z difference of 100 Da"
- [readme] modified cosine as field standard: [evidence span withheld — anchor status: external_doc]
