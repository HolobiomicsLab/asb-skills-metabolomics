---
name: r-runtime-and-memory-profiling
description: Use when benchmarking an R network-construction function (e.g. MetaNet's
  c_net_calculate) across a series of feature sizes to characterize runtime and memory
  scaling against stated performance targets.
when_to_use_negative:
- The goal is to compare MetaNet's accuracy or network topology against other tools
  rather than measure its computational cost
- The dataset has a single fixed size and no scaling comparison is needed — a single
  system.time run suffices, not a scaling-profile skill
- The workload is not correlation-based network construction (e.g. module detection,
  layout, or visualization), whose scaling may differ from the ~quadratic regime validated
  here
tools:
- name: MetaNet
  role: Benchmarked package whose c_net_calculate correlation step is timed and memory-profiled
    across feature sizes
  repo: github.com/Asa12138/MetaNet
- name: pcutils
  role: Companion R package providing the otutab test data used to build the synthetic
    benchmark inputs
  repo: github.com/Asa12138/pcutils
- name: igraph
  role: Loaded as the underlying network library on which MetaNet's benchmarked construction
    functions operate
- name: R
  role: Runtime environment providing system.time timing and gc()-based memory profiling
    for the benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:8baed71f251a891ab3c3cbf02942636d9af75a8d156538688c4dfd0cbda86f1c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaNet
  - R
  - igraph
  - pcutils
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# r-runtime-and-memory-profiling

## Summary

Benchmarks R-based correlation network construction by timing runs with system.time and recording peak memory via gc()/memory profiling across increasing feature counts, then fitting log-log regressions to recover the runtime/memory scaling exponent.

## When to use

Use when benchmarking an R network-construction function (e.g. MetaNet's c_net_calculate) across a series of feature sizes to characterize runtime and memory scaling against stated performance targets.

## When NOT to use

- The goal is to compare MetaNet's accuracy or network topology against other tools rather than measure its computational cost
- The dataset has a single fixed size and no scaling comparison is needed — a single system.time run suffices, not a scaling-profile skill
- The workload is not correlation-based network construction (e.g. module detection, layout, or visualization), whose scaling may differ from the ~quadratic regime validated here

## Inputs

- Seeded synthetic omics count tables (e.g. totu, totu2) transposed to features-as-columns, at graded feature sizes from ~100 to ~5000 features
- Fixed correlation-method configuration: Pearson/Spearman via c_net_calculate with p.adjust.method/p.adjust.mode specified identically for all sizes

## Outputs

- benchmark_results.csv with per-size elapsed time, peak memory, and fitted log-log scaling exponents
- scaling_plot.png showing runtime and memory curves versus feature number (Figure 1D/1E style)

## How to apply

Install MetaNet and its companion pcutils from GitHub via devtools::install_github("Asa12138/pcutils") and devtools::install_github("Asa12138/MetaNet"), loading igraph alongside. Generate seeded synthetic omics count tables (totu, totu2) at a ladder of feature sizes spanning the target regime and beyond it (e.g. 100, 250, 500, 750, 1000, 2500, 5000 features). For each size, call c_net_calculate with the correlation method (Pearson/Spearman) and p.adjust.method/p.adjust.mode held identical across sizes, timing each run with system.time and recording peak memory with gc()/R memory profiling. Verify that runs on datasets with fewer than 1,000 features complete in under 0.2 seconds using less than 100 MB of memory. Finally, fit a log-log linear regression of elapsed time and peak memory against feature number and confirm the fitted exponent is approximately 2 (quadratic scaling), consistent with the Figure 1D/1E benchmark curves.

## Related tools

- **MetaNet** (Benchmarked package whose c_net_calculate correlation step is timed and memory-profiled across feature sizes) — github.com/Asa12138/MetaNet
- **pcutils** (Companion R package providing the otutab test data used to build the synthetic benchmark inputs) — github.com/Asa12138/pcutils
- **igraph** (Loaded as the underlying network library on which MetaNet's benchmarked construction functions operate)
- **R** (Runtime environment providing system.time timing and gc()-based memory profiling for the benchmark)

## Examples

```
system.time(cor <- c_net_calculate(totu, method = "spearman", p.adjust.method = "BH"))
```

## Evaluation signals

- All runs on datasets with fewer than 1,000 features complete in under 0.2 seconds of elapsed time
- All runs on datasets with fewer than 1,000 features use less than 100 MB peak memory
- Log-log linear regression of elapsed time vs. feature number yields an exponent of approximately 2 (quadratic scaling)
- Log-log regression of peak memory vs. feature number yields an exponent of approximately 2
- Correlation method and thresholding settings are verifiably identical across all benchmarked sizes (no confounding parameter changes)

## Limitations

- The under-0.2 s / under-100 MB targets apply specifically to datasets with fewer than 1,000 features; behavior above 1,000 features is extrapolated via the fitted exponent
- Benchmark claims (up to 100-fold speed, 50-fold memory reduction) are relative to other R packages and depend on hardware not specified in the article
- Article sections lack version numbers for MetaNet and compared tools, and no changelog is available, so benchmark reproducibility across package versions is not guaranteed

## Evidence

- [readme] optimized vectorized matrix algorithms reduce memory and runtime: [evidence span withheld — anchor status: external_doc]
