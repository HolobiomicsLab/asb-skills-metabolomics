---
name: runtime-scaling-benchmarking
description: Use when benchmarking the computational scalability (runtime and memory)
  of a spectrum clustering or data-processing tool across progressively larger dataset
  subsets to verify a claimed linear scaling trend.
when_to_use_negative:
- When the goal is to compare clustering quality (purity, completeness, incorrectly
  clustered spectra) rather than computational performance — the paper uses separate
  quality-evaluation workflows for that.
- When the input is a single small peak file, since scaling behavior cannot be assessed
  without multiple progressively larger subsets.
- When benchmarking tools that were not designed for large-scale spectra clustering
  (e.g. single-file spectrum viewers), where a runtime scaling claim is not applicable.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: Spectrum clustering tool whose runtime is benchmarked across dataset subsets;
    invoked with precursor_tol 20 ppm, fragment_tol 0.05, eps 0.10
  repo: https://github.com/bittremieux/falcon
- name: Python 3.8
  role: Runtime environment used to implement/run falcon and to tabulate and plot
    runtime-vs-spectra results (matplotlib, CSV output)
  repo: https://github.com/bittremieux/falcon_notebooks
- name: ThermoRawFileParser 1.2.3
  role: Converts the PRIDE PXD000561 raw files to the MGF peak files used as scaling-benchmark
    inputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:0fa3591c11741f1854cee1b27eb339d46cea1c097d274dc0154d3ce80c8a7a54
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Python 3.8
  - ThermoRawFileParser 1.2.3
  - falcon
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# runtime-scaling-benchmarking

## Summary

Benchmarks how a tool's runtime scales with input size by running falcon on progressively larger random subsets of the Kim et al. human draft proteome MGF peak files (10 to 2212 files, ~25 million spectra) and tabulating/plotting runtime versus number of spectra. It reproduces the paper's evaluation showing falcon exhibits optimal near-linear runtime scaling despite quadratic pairwise spectrum comparisons.

## When to use

Use when benchmarking the computational scalability (runtime and memory) of a spectrum clustering or data-processing tool across progressively larger dataset subsets to verify a claimed linear scaling trend.

## When NOT to use

- When the goal is to compare clustering quality (purity, completeness, incorrectly clustered spectra) rather than computational performance — the paper uses separate quality-evaluation workflows for that.
- When the input is a single small peak file, since scaling behavior cannot be assessed without multiple progressively larger subsets.
- When benchmarking tools that were not designed for large-scale spectra clustering (e.g. single-file spectrum viewers), where a runtime scaling claim is not applicable.

## Inputs

- Converted MGF peak files from the human draft proteome dataset by Kim et al. (PRIDE PXD000561; raw files from PRIDE converted using ThermoRawFileParser 1.2.3)
- Random subsets of 10 to 2212 peak files (up to ~25 million MS/MS spectra)
- Multi-core compute node (e.g. two 14-core Intel E5-2680v4 CPUs, 128GB memory)

## Outputs

- scaling_runtime.csv with columns: subset size, number of spectra, runtime, peak memory per falcon run
- Matplotlib plot of runtime versus number of input spectra showing the scaling behavior

## How to apply

Split the converted MGF peak files of the Kim et al. human draft proteome dataset (PRIDE PXD000561) into random subsets of 10, 50, 100, 500, 1000, 1500, 2000, and 2212 files. For each subset, run falcon with the paper's clustering settings (precursor tolerance 20 ppm, fragment tolerance 0.05 Da, cosine distance threshold eps 0.10) and record total runtime and peak memory consumption per run. Tabulate subset size, number of spectra, runtime, and memory into a CSV file (scaling_runtime.csv) using Python 3.8. Plot runtime versus number of input spectra across the subsets with matplotlib and compare the observed curve against the paper's reported near-linear scaling. Rationale: falcon avoids exhaustive pairwise comparison via feature hashing, inverted-index nearest neighbor searching, and DBSCAN density-based clustering, so its runtime should grow near-linearly with spectra count rather than quadratically.

## Related tools

- **falcon** (Spectrum clustering tool whose runtime is benchmarked across dataset subsets; invoked with precursor_tol 20 ppm, fragment_tol 0.05, eps 0.10) — https://github.com/bittremieux/falcon
- **Python 3.8** (Runtime environment used to implement/run falcon and to tabulate and plot runtime-vs-spectra results (matplotlib, CSV output)) — https://github.com/bittremieux/falcon_notebooks
- **ThermoRawFileParser 1.2.3** (Converts the PRIDE PXD000561 raw files to the MGF peak files used as scaling-benchmark inputs)

## Examples

```
falcon peak/*.mgf falcon --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- The runtime-versus-spectra plot reproduces the near-linear scaling trend reported in the paper's scaling evaluation rather than a quadratic trend.
- The 2212-file run (~25 million spectra) completes within the order of hours on a multi-core compute node (e.g. two 14-core Intel E5-2680v4 CPUs, 128GB memory).
- scaling_runtime.csv contains a monotonic runtime column for all eight subset sizes (10, 50, 100, 500, 1000, 1500, 2000, 2212 files) with recorded peak memory.
- All runs use identical clustering settings (precursor_tol 20 ppm, fragment_tol 0.05, eps 0.10) so runtime differences reflect dataset size only.
- Subset assignment is random and non-overlapping so increasing subset sizes represent progressively larger samples of the same dataset.

## Limitations

- Pairwise spectrum comparisons scale quadratically with the number of spectra; the observed linear runtime relies on falcon's nearest neighbor indexing optimizations and may not transfer to exhaustively comparing tools.
- Changing falcon's nearest neighbor indexing settings (n_probe, n_neighbors, n_neighbors_ann, low_dim) tunes clustering time versus accuracy and would confound the scaling measurement.
- The eps threshold depends on the spectral characteristics of the data and any spectrum preprocessing (e.g. scaling of peak intensities), so the benchmark result is dataset- and setting-specific.

## Evidence
