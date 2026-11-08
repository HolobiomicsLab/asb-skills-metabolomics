---
name: compute-performance-visualization
description: 'Use when you must verify a tool''s compute scaling: clustering progressively
  larger MGF subsets and plotting runtime versus number of spectra to check for the
  near-linear trend.'
when_to_use_negative:
- Evaluating clustering quality (cluster purity, completeness, incorrectly clustered
  spectra) rather than compute performance — use the clustering-quality benchmark
  instead
- Only one or a few peak files are available, so no subset series can be formed to
  estimate a scaling curve
- Profiling a tool that exhaustively performs all pairwise spectrum comparisons, where
  quadratic scaling is expected and a linear-trend validation is meaningless
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3810
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: Spectrum clustering tool benchmarked across subsets of 10–2212 MGF peak files;
    run per subset with precursor_tol 20 ppm, fragment_tol 0.05 Da, eps 0.10 while
    timing runtime and memory
  repo: https://github.com/bittremieux/falcon
- name: Python 3.8
  role: Runtime environment in which falcon was implemented; used for the scaling
    runs, timing capture, and plotting scripts
  repo: https://github.com/bittremieux/falcon
- name: matplotlib
  role: Plots runtime versus number of input spectra across the subsets to visualize
    the scaling behavior
- name: ThermoRawFileParser
  role: Upstream conversion (version 1.2.3) of raw files from PRIDE PXD000561 into
    the MGF peak files that are split into scaling subsets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:a1a7173c61527346a953805a28815f18b04a3eea90ba1fe83bb60f28265d33db
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Python 3.8
  - ThermoRawFileParser
  - falcon
  - matplotlib
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3810
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# compute-performance-visualization

## Summary

Benchmarks how a tool's runtime and peak memory scale with input size by clustering progressively larger random subsets of MGF peak files (10 to 2212 files, ~25 million spectra) and plotting runtime versus number of input spectra. Here it validates falcon's near-linear runtime scaling even though pairwise spectrum comparisons scale quadratically.

## When to use

Use when you must verify a tool's compute scaling: clustering progressively larger MGF subsets and plotting runtime versus number of spectra to check for the near-linear trend.

## When NOT to use

- Evaluating clustering quality (cluster purity, completeness, incorrectly clustered spectra) rather than compute performance — use the clustering-quality benchmark instead
- Only one or a few peak files are available, so no subset series can be formed to estimate a scaling curve
- Profiling a tool that exhaustively performs all pairwise spectrum comparisons, where quadratic scaling is expected and a linear-trend validation is meaningless

## Inputs

- Converted MGF peak files from the Kim et al. human draft proteome dataset (PRIDE PXD000561, ~25 million spectra)
- Random subsets of 10 to 2212 peak files drawn from the dataset
- Fixed falcon settings: precursor_tol 20 ppm, fragment_tol 0.05 Da, cosine distance threshold eps 0.10

## Outputs

- scaling_runtime.csv tabulating subset size, number of spectra, total runtime, and peak memory per run
- matplotlib plot of runtime versus number of input spectra showing the scaling curve
- Per-run resource measurements (runtime, peak memory) for each subset size

## How to apply

Split the converted MGF peak files of the Kim et al. human draft proteome dataset (PRIDE PXD000561, converted with ThermoRawFileParser 1.2.3) into random subsets of 10, 50, 100, 500, 1000, 1500, 2000, and 2212 files. Run falcon on each subset with fixed paper settings — precursor_tol 20 ppm, fragment_tol 0.05 Da, cosine distance threshold eps 0.10 — recording total runtime and peak memory per run so only input size varies between runs. Tabulate subset size, number of spectra, runtime, and memory in a CSV (scaling_runtime.csv), then plot runtime versus number of input spectra with matplotlib. The rationale: falcon's feature hashing, Voronoi-diagram nearest neighbor indexes, and DBSCAN clustering avoid exhaustive all-vs-all spectrum comparisons, so the observed curve should be near-linear rather than quadratic. Validate that the largest run (2212 files, ~25 million spectra) completes within the order of hours on a multi-core node such as two 14-core Intel E5-2680v4 CPUs with 128 GB memory.

## Related tools

- **falcon** (Spectrum clustering tool benchmarked across subsets of 10–2212 MGF peak files; run per subset with precursor_tol 20 ppm, fragment_tol 0.05 Da, eps 0.10 while timing runtime and memory) — https://github.com/bittremieux/falcon
- **Python 3.8** (Runtime environment in which falcon was implemented; used for the scaling runs, timing capture, and plotting scripts) — https://github.com/bittremieux/falcon
- **matplotlib** (Plots runtime versus number of input spectra across the subsets to visualize the scaling behavior)
- **ThermoRawFileParser** (Upstream conversion (version 1.2.3) of raw files from PRIDE PXD000561 into the MGF peak files that are split into scaling subsets)

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- Runtime-versus-spectra plot reproduces a near-linear scaling trend across the 10→2212-file subsets instead of a quadratic curve
- scaling_runtime.csv contains one row per subset size (10, 50, 100, 500, 1000, 1500, 2000, 2212) with columns for spectra count, runtime, and peak memory
- All runs use identical falcon settings (precursor_tol 20 ppm, fragment_tol 0.05 Da, eps 0.10), so runtime differences are attributable to input size alone
- The 2212-file run (~25 million spectra) completes within the order of hours on a multi-core reference node (e.g. two 14-core Intel E5-2680v4 CPUs, 128 GB memory)
- Peak memory recorded per run stays within the reference node's 128 GB memory budget

## Limitations

- Linear scaling is a property of falcon's nearest neighbor indexing; alternative clustering tools (MS-Cluster, spectra-cluster, MaRaCluster, msCRUSH) may exhibit different scaling behavior
- The README warns that nearest neighbor indexing settings (n_probe, n_neighbors, n_neighbors_ann, low_dim) 'can be modified to tune clustering time versus accuracy' — changing them mid-experiment confounds the runtime comparison
- Validation of the ~25-million-spectra run completing in hours assumes multi-core hardware with large memory; smaller nodes may not finish the 2212-file run or hold the peak memory requirement

## Evidence

- [methods] PRIDE download: [evidence span withheld — anchor status: not_found]
