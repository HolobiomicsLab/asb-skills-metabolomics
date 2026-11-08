---
name: computational-runtime-measurement
description: Use when comparing algorithmic efficiency of spectrum clustering tools
  (e.g. falcon vs msCRUSH vs brute-force pairwise comparison) and you need concrete
  runtime and per-spectrum comparison counts on a large public MS/MS dataset.
when_to_use_negative:
- When the goal is clustering quality (cluster purity/completeness) rather than computational
  cost — runtime measurement does not validate that eps 0.05 yields ~1% incorrectly
  clustered spectra on its own
- When the target dataset is metabolomics or top-down data, where falcon's default
  preprocessing settings (min_peaks 5, min_mz_range 250 Da) need adjustment and runtime
  figures will not transfer
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: Spectrum clustering tool whose nearest-neighbor searching is instrumented
    to count comparisons and measure runtime
  repo: https://github.com/bittremieux/falcon
- name: msCRUSH
  role: Alternative spectrum clustering tool run on identical data as a runtime comparison
    point
- name: ThermoRawFileParser
  role: Converts vendor raw files (version 1.2.3) to MGF format as the benchmark input
- name: Python 3.8
  role: Language in which falcon was implemented, used for instrumenting the run
  repo: https://github.com/bittremieux/falcon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:22f52a1c0c04f0ff314f8b82e8a4a4e04d6cb8d2bbbf343c1856f393d8841284
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# computational-runtime-measurement

## Summary

Benchmark the computational cost of a spectrum clustering pipeline by instrumenting total wall-clock runtime and counting executed spectrum–spectrum comparisons, using falcon's nearest neighbor searching on the Kim et al. draft human proteome dataset as the reference workload.

## When to use

Use when comparing algorithmic efficiency of spectrum clustering tools (e.g. falcon vs msCRUSH vs brute-force pairwise comparison) and you need concrete runtime and per-spectrum comparison counts on a large public MS/MS dataset.

## When NOT to use

- When the goal is clustering quality (cluster purity/completeness) rather than computational cost — runtime measurement does not validate that eps 0.05 yields ~1% incorrectly clustered spectra on its own
- When the target dataset is metabolomics or top-down data, where falcon's default preprocessing settings (min_peaks 5, min_mz_range 250 Da) need adjustment and runtime figures will not transfer

## Inputs

- 2212 raw LC-MS/MS files of the Kim et al. draft human proteome dataset (PRIDE PXD000561)
- MGF-format peak files converted with ThermoRawFileParser 1.2.3
- falcon clustering configuration: precursor_tol 20 ppm, fragment_tol 0.05 Da, eps 0.05
- msCRUSH configuration: 100 clustering iterations, 15 hash functions per hash table

## Outputs

- runtime_benchmark.csv with falcon wall-clock runtime (~5 hours), average spectrum–spectrum comparisons per spectrum (~47), and msCRUSH runtime (~4 hours)
- Comparison count per spectrum for nearest-neighbor searching vs brute-force pairwise comparison within the 20 ppm precursor tolerance

## How to apply

Obtain the 2212 raw files of the Kim et al. draft human proteome dataset (PRIDE PXD000561) and convert them to MGF with ThermoRawFileParser 1.2.3. Run falcon with precursor_tol 20 ppm, fragment_tol 0.05 Da, and eps 0.05 (the configuration yielding ~1% incorrectly clustered spectra) on a two 14-core Intel E5-2680v4 node with 128GB memory, using all cores, and instrument the run to record total wall-clock runtime and count spectrum–spectrum comparisons during nearest-neighbor searching (~5 hours, ~47 comparisons per spectrum on average). Run msCRUSH (100 clustering iterations, 15 hash functions per hash table) on the same data as a context point (~4 hours). Compile the measurements into a runtime_benchmark.csv and compare the ANN-based comparison count against exhaustive pairwise comparison within the 20 ppm precursor mass tolerance.

## Related tools

- **falcon** (Spectrum clustering tool whose nearest-neighbor searching is instrumented to count comparisons and measure runtime) — https://github.com/bittremieux/falcon
- **msCRUSH** (Alternative spectrum clustering tool run on identical data as a runtime comparison point)
- **ThermoRawFileParser** (Converts vendor raw files (version 1.2.3) to MGF format as the benchmark input)
- **Python 3.8** (Language in which falcon was implemented, used for instrumenting the run) — https://github.com/bittremieux/falcon

## Examples

```
falcon peak/*.mgf falcon --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.05
```

## Evaluation signals

- Recorded falcon total runtime is on the order of the reported ~5 hours on the two 14-core Intel E5-2680v4 / 128GB node
- Average spectrum–spectrum comparisons per spectrum during nearest-neighbor searching is approximately 47, not thousands (brute-force would compare all spectra within the 20 ppm precursor tolerance)
- Recorded msCRUSH runtime (100 iterations, 15 hash functions) is approximately 4 hours on the same hardware
- runtime_benchmark.csv contains one row per tool with runtime and comparison-count columns populated for all 2212 files' worth of data

## Limitations

- Runtime figures are hardware-dependent: they were measured on a node with two 14-core Intel E5-2680v4 CPUs and 128GB memory using all available cores
- falcon's nearest-neighbor search trades slight accuracy for speed; tuning n_probe, n_neighbors, and low_dim changes both runtime and clustering accuracy
- Comparison counts reflect falcon's approximate nearest-neighbor search and are only meaningful relative to exhaustive pairwise comparison within the same precursor mass tolerance

## Evidence

- [results] ann comparisons vs brute force: [evidence span withheld — anchor status: not_found]
