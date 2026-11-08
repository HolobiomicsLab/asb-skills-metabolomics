---
name: nearest-neighbor-search-comparison-profiling
description: Use when benchmarking whether ANN indexing (feature hashing + Voronoi
  inverted index) avoids exhaustive pairwise spectrum comparison at scale, e.g. on
  millions of MS/MS spectra from proteomics PRIDE datasets.
when_to_use_negative:
- When the goal is clustering quality assessment rather than profiling the computational
  cost of nearest neighbor searching (use falcon_notebooks' tool-comparison notebooks
  instead).
- 'When clustering metabolomics or top-down data: falcon''s default preprocessing
  (min_peaks 5, min_mz_range 250 m/z, min_mz 101, max_mz 500 m/z) must first be adjusted,
  invalidating cost comparability with the proteomics benchmark.'
- When the dataset is too small for ANN to matter — exhaustive pairwise comparison
  within the precursor tolerance may be feasible and the ~47 comparisons/spectrum
  benchmark would not be meaningful.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: falcon
  role: Spectrum clustering tool whose nearest neighbor searching is instrumented
    to count spectrum–spectrum comparisons and record runtime
  repo: https://github.com/bittremieux/falcon
- name: msCRUSH
  role: Alternative spectrum clustering tool run as a runtime baseline on the same
    dataset
- name: ThermoRawFileParser
  role: Converts Thermo raw files to MGF peak list files before clustering
- name: Python
  role: falcon's implementation language (Python 3.8), used to instrument the run
    for comparison counting and timing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:c62de9a9fedf1e8de413389b27aec1c42b49ac79d97e5d62d73189ecda4cc0a0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Python
  - ThermoRawFileParser
  - falcon
  - msCRUSH
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# nearest-neighbor-search-comparison-profiling

## Summary

Profiles the computational cost of falcon's approximate nearest neighbor (ANN) searching by instrumenting a clustering run to count spectrum–spectrum comparisons per spectrum, contrasting it with exhaustive pairwise comparison within the 20 ppm precursor tolerance and with msCRUSH's runtime as a baseline.

## When to use

Use when benchmarking whether ANN indexing (feature hashing + Voronoi inverted index) avoids exhaustive pairwise spectrum comparison at scale, e.g. on millions of MS/MS spectra from proteomics PRIDE datasets.

## When NOT to use

- When the goal is clustering quality assessment rather than profiling the computational cost of nearest neighbor searching (use falcon_notebooks' tool-comparison notebooks instead).
- When clustering metabolomics or top-down data: falcon's default preprocessing (min_peaks 5, min_mz_range 250 m/z, min_mz 101, max_mz 500 m/z) must first be adjusted, invalidating cost comparability with the proteomics benchmark.
- When the dataset is too small for ANN to matter — exhaustive pairwise comparison within the precursor tolerance may be feasible and the ~47 comparisons/spectrum benchmark would not be meaningful.

## Inputs

- Thermo raw files of a large MS/MS dataset (e.g. PRIDE PXD000561 draft human proteome, 25 million spectra)
- MGF-format peak list files converted from the raw files
- falcon clustering configuration (precursor_tol 20 ppm, fragment_tol 0.05 Da, eps 0.05)
- msCRUSH configuration (100 clustering iterations, 15 hash functions per hash table)

## Outputs

- runtime_benchmark.csv with falcon wall-clock runtime, average spectrum–spectrum comparisons per spectrum, and msCRUSH runtime
- falcon cluster assignments (comma-separated file mapping each MS/MS spectrum to its cluster label)

## How to apply

Obtain the raw files (e.g. the 2212 raw files of the Kim et al. draft human proteome dataset, PRIDE PXD000561) and convert them to MGF with ThermoRawFileParser (version 1.2.3). Run falcon with precursor_tol 20 ppm, fragment_tol 0.05 Da, and eps 0.05 (the configuration yielding approximately 1% incorrectly clustered spectra) on all available cores of a multi-core node (e.g. two 14-core Intel E5-2680v4 CPUs, 128GB memory). Instrument the run to count the spectrum–spectrum comparisons executed during nearest neighbor searching — falcon builds the sparse pairwise distance matrix from a Voronoi diagram inverted index without exhaustively comparing all spectra — and record total wall-clock runtime. Run msCRUSH (100 clustering iterations, 15 hash functions per hash table) on the same data as a baseline. Compile total runtimes and average comparisons per spectrum (expected ~5 h and ~47 comparisons per spectrum for falcon, ~4 h for msCRUSH) into a benchmark CSV; the key rationale is that the ANN index restricts comparisons to a limited number of nearest neighbors within the precursor mass tolerance rather than all pairwise spectra.

## Related tools

- **falcon** (Spectrum clustering tool whose nearest neighbor searching is instrumented to count spectrum–spectrum comparisons and record runtime) — https://github.com/bittremieux/falcon
- **msCRUSH** (Alternative spectrum clustering tool run as a runtime baseline on the same dataset)
- **ThermoRawFileParser** (Converts Thermo raw files to MGF peak list files before clustering)
- **Python** (falcon's implementation language (Python 3.8), used to instrument the run for comparison counting and timing)

## Examples

```
falcon peak/*.mgf falcon --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.05
```

## Evaluation signals

- Recorded falcon total runtime is on the order of the reported ~5 hours on the full 2212-file PXD000561 dataset
- Average spectrum–spectrum comparisons during nearest neighbor searching is approximately 47 per spectrum, far below exhaustive pairwise comparison within the 20 ppm precursor tolerance
- Recorded msCRUSH runtime is approximately 4 hours on the same data and hardware
- The falcon run at eps 0.05 produces approximately 1% incorrectly clustered spectra, confirming the benchmarked configuration matches the paper's setting
- Cluster assignment output is a CSV with exactly one line per MS/MS spectrum containing its cluster label

## Limitations

- The accuracy/speed trade-off of nearest neighbor searching depends on n_probe, n_neighbors, and low_dim; inspecting fewer lists or querying fewer neighbors runs faster but gives slightly less accurate clustering.
- The eps cosine distance threshold crucially governs cluster purity, and the scaling method can influence the eps threshold.
- falcon's default spectrum preprocessing targets bottom-up proteomics; metabolomics or top-down data need adjusted settings.
- Changing nearest neighbor indexing settings is only recommended for advanced users, and only Linux and OSX platforms are supported (Python 3.8+).

## Evidence
