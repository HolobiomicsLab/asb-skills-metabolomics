---
name: clustering-tool-performance-evaluation
description: Use when comparing the computational efficiency of spectrum clustering
  tools (falcon, msCRUSH, MS-Cluster, MaRaCluster, spectra-cluster) on the same raw
  MS/MS dataset, specifically to quantify comparisons per spectrum and runtime at
  a fixed clustering quality point.
when_to_use_negative:
- When the goal is clustering quality (purity/completeness) rather than computational
  cost — use the incorrect-clustering-rate evaluation instead
- When the dataset is small enough that brute-force pairwise comparison is tractable,
  since nearest-neighbor speedups (~47 comparisons/spectrum) will not be meaningful
- When clustering metabolomics or top-down data, where falcon's default proteomics
  preprocessing (min 5 peaks, 250 m/z range, 101–500 m/z window) and eps need adjustment
  before any benchmark is fair
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: falcon
  role: Tool being benchmarked; run with precursor_tol 20 ppm, fragment_tol 0.05 Da,
    eps 0.05 to measure runtime and nearest-neighbor comparison counts
  repo: https://github.com/bittremieux/falcon
- name: msCRUSH
  role: Alternative clustering tool run as a runtime context point (100 iterations,
    15 hash functions per hash table)
- name: ThermoRawFileParser
  role: Converts the raw Thermo files to MGF format prior to clustering (version 1.2.3
    used in the paper)
- name: Python 3.8
  role: Implementation environment for falcon and for instrumenting the run to count
    comparisons and record wall-clock time
  repo: https://github.com/bittremieux/falcon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:fa6d842b5dd23a3064fba5928e325e57e89f28823490e99698db49cad48cc6b0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Python
  - Python 3.8
  - ThermoRawFileParser
  - falcon
  - msCRUSH
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# clustering-tool-performance-evaluation

## Summary

Benchmarks MS/MS spectrum clustering tools (e.g. falcon vs msCRUSH) by instrumenting runs on a shared large-scale dataset to record total wall-clock runtime and the number of spectrum-spectrum comparisons performed per spectrum, quantifying how much the nearest-neighbor indexing avoids exhaustive pairwise comparison.

## When to use

Use when comparing the computational efficiency of spectrum clustering tools (falcon, msCRUSH, MS-Cluster, MaRaCluster, spectra-cluster) on the same raw MS/MS dataset, specifically to quantify comparisons per spectrum and runtime at a fixed clustering quality point.

## When NOT to use

- When the goal is clustering quality (purity/completeness) rather than computational cost — use the incorrect-clustering-rate evaluation instead
- When the dataset is small enough that brute-force pairwise comparison is tractable, since nearest-neighbor speedups (~47 comparisons/spectrum) will not be meaningful
- When clustering metabolomics or top-down data, where falcon's default proteomics preprocessing (min 5 peaks, 250 m/z range, 101–500 m/z window) and eps need adjustment before any benchmark is fair

## Inputs

- Raw MS/MS data files from a public repository (e.g. 2212 Thermo raw files from PRIDE project PXD000561)
- MGF-format peak files converted from the raw data
- Tool configuration parameters (precursor tolerance, fragment tolerance, clustering threshold, number of hash functions/iterations)

## Outputs

- runtime_benchmark.csv containing per-tool total wall-clock runtime
- Average number of spectrum-spectrum comparisons per spectrum during nearest-neighbor searching
- Cluster assignments (e.g. falcon.csv) and optionally representative consensus spectra (MGF) for downstream quality checks

## How to apply

Obtain the raw files of a reference dataset (e.g. the 2212 raw files of the Kim et al. draft human proteome dataset, PRIDE PXD000561) and convert them to MGF with ThermoRawFileParser. Run each tool on the full dataset with identical, documented parameters — for falcon use precursor_tol 20 ppm, fragment_tol 0.05 Da, and eps 0.05 (the cosine distance threshold yielding approximately 1% incorrectly clustered spectra), using all available cores on a specified compute node (two 14-core Intel E5-2680v4 CPUs, 128 GB memory). Instrument the run to count total spectrum-spectrum comparisons executed during nearest-neighbor searching and divide by the number of spectra to get average comparisons per spectrum. Run the comparison tool (msCRUSH with 100 clustering iterations and 15 hash functions per hash table) under the same hardware conditions as a runtime context point. Compile runtime and comparisons-per-spectrum into a CSV and validate against reported figures (~5 h falcon, ~47 comparisons/spectrum, ~4 h msCRUSH).

## Related tools

- **falcon** (Tool being benchmarked; run with precursor_tol 20 ppm, fragment_tol 0.05 Da, eps 0.05 to measure runtime and nearest-neighbor comparison counts) — https://github.com/bittremieux/falcon
- **msCRUSH** (Alternative clustering tool run as a runtime context point (100 iterations, 15 hash functions per hash table))
- **ThermoRawFileParser** (Converts the raw Thermo files to MGF format prior to clustering (version 1.2.3 used in the paper))
- **Python 3.8** (Implementation environment for falcon and for instrumenting the run to count comparisons and record wall-clock time) — https://github.com/bittremieux/falcon

## Examples

```
falcon peak/*.mgf falcon --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.05 --n_jobs -1
```

## Evaluation signals

- falcon total runtime on the full 2212-file dataset is on the order of ~5 hours on a two 14-core Intel E5-2680v4 node with 128 GB memory
- Instrumented comparison count averages approximately 47 spectrum-spectrum comparisons per spectrum, far below the number of precursor-mass-tolerance neighbors under exhaustive comparison
- msCRUSH runtime on the identical data and hardware is approximately 4 hours
- The falcon configuration (eps 0.05) corresponds to the reported ~1% incorrectly clustered spectra, confirming efficiency is measured at an equivalent quality operating point
- All tools are run with all available processor cores on the same node so runtimes are directly comparable

## Limitations

- Runtime is hardware- and core-count-dependent; the reported ~5 h (falcon) and ~4 h (msCRUSH) figures only transfer to comparable compute nodes run with all cores
- Comparisons-per-spectrum is measured for falcon's approximate nearest-neighbor search and can shift if n_probe, n_neighbors, n_neighbors_ann, or low_dim settings are changed
- The benchmark reflects efficiency at a single clustering-quality operating point (eps 0.05, ~1% incorrectly clustered); other tools' speed may scale differently at their own quality settings

## Evidence

- [results] Using its advanced nearest neighbor searching approach, falcon required on average only 47 spectrum-spectrum comparisons per spectrum, instead of exhaustively comparing all spectra to each other.: [evidence span withheld — anchor status: not_found]
- [methods] Raw data provenance and conversion: [evidence span withheld — anchor status: not_found]
