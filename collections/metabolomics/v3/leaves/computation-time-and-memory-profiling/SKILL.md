---
name: computation-time-and-memory-profiling
description: Use when comparing computational efficiency of network-construction tools
  on feature abundance matrices (e.g., 50–1,000 features) and you need defensible
  median elapsed-time and memory measurements per package.
when_to_use_negative:
- When the goal is to evaluate network quality or biological validity (topology, module
  detection) rather than computational efficiency
- When comparing tools whose correlation methods or significance thresholds cannot
  be aligned, making timing comparisons non-equivalent
- When the dataset scale differs fundamentally from the benchmarked regime (e.g.,
  non-correlation-based or multimillion-feature workloads where memory may thrash
  rather than scale linearly)
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaNet
  role: Tool under benchmark; its c_net_calculate() correlation-based network construction
    is timed against comparators
  repo: https://github.com/Asa12138/MetaNet
- name: R
  role: Implementation environment for generating synthetic matrices, running all
    tools, and profiling with bench::mark
- name: WGCNA
  role: Comparator package; its corAndPvalue call is benchmarked against MetaNet on
    identical inputs
- name: microeco
  role: Comparator package; its trans_network$new constructor is benchmarked
- name: ggClusterNet
  role: Comparator package; its corMicro call is benchmarked
- name: NetCoMi
  role: Comparator package; its netConstruct call is benchmarked
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:04dfe2592e77335e9f2c57c1a598a25236e0765ec7371a9a0a5895adc549ebcd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Asa12138/MetaNet
  tools:
  - MetaNet
  - NetCoMi
  - R
  - WGCNA
  - ggClusterNet
  - microeco
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# computation-time-and-memory-profiling

## Summary

Benchmark the runtime and memory footprint of correlation-based network construction across competing R packages by timing identical calls on shared synthetic inputs with bench::mark, quantifying speed-ups such as MetaNet's up-to-100-fold time and 50-fold memory advantages over WGCNA, microeco, ggClusterNet, and NetCoMi.

## When to use

Use when comparing computational efficiency of network-construction tools on feature abundance matrices (e.g., 50–1,000 features) and you need defensible median elapsed-time and memory measurements per package.

## When NOT to use

- When the goal is to evaluate network quality or biological validity (topology, module detection) rather than computational efficiency
- When comparing tools whose correlation methods or significance thresholds cannot be aligned, making timing comparisons non-equivalent
- When the dataset scale differs fundamentally from the benchmarked regime (e.g., non-correlation-based or multimillion-feature workloads where memory may thrash rather than scale linearly)

## Inputs

- Synthetic feature abundance count matrices at 50, 100, 200, 400, and 1,000 features (fixed random seed, identical across tools)
- R environment with MetaNet, pcutils, WGCNA, microeco, ggClusterNet, and NetCoMi installed

## Outputs

- bench::mark measurements of elapsed time and memory allocation per package per dataset size (20 repetitions)
- Summary table of median computation time and memory per package per dataset size (benchmark_results.csv)

## How to apply

Generate synthetic feature abundance count matrices at several sizes (here 50, 100, 200, 400, and 1,000 features) with a fixed random seed so every tool receives identical inputs. Install MetaNet and its pcutils dependency via devtools::install_github("Asa12138/pcutils") and devtools::install_github("Asa12138/MetaNet"), and load comparators WGCNA, microeco, ggClusterNet, and NetCoMi in R. Run each tool's network-construction entry point on each matrix — c_net_calculate(totu, totu2, method, p.adjust.method, threads, ...) for MetaNet, corAndPvalue for WGCNA, trans_network$new for microeco, corMicro for ggClusterNet, and netConstruct for NetCoMi — keeping the correlation method and significance filtering identical across tools. Time each call with bench::mark using 20 repetitions on a defined hardware platform (Apple M2, macOS), capturing elapsed time and memory allocation, then aggregate medians per package per dataset size into a summary table (benchmark_results.csv). Validate by confirming MetaNet's runtime advantage is consistent with the reported up-to-100-fold speed and up-to-50-fold memory improvements, with sub-0.2-second and sub-100 MB performance on datasets under 1,000 features.

## Related tools

- **MetaNet** (Tool under benchmark; its c_net_calculate() correlation-based network construction is timed against comparators) — https://github.com/Asa12138/MetaNet
- **R** (Implementation environment for generating synthetic matrices, running all tools, and profiling with bench::mark)
- **WGCNA** (Comparator package; its corAndPvalue call is benchmarked against MetaNet on identical inputs)
- **microeco** (Comparator package; its trans_network$new constructor is benchmarked)
- **ggClusterNet** (Comparator package; its corMicro call is benchmarked)
- **NetCoMi** (Comparator package; its netConstruct call is benchmarked)

## Examples

```
library(bench); res <- bench::mark(cor = c_net_calculate(totu), corAndPvalue(totu), netConstruct(totu), iterations = 20, memory = TRUE)
```

## Evaluation signals

- Each package receives bit-identical input matrices (same fixed random seed and dataset sizes of 50/100/200/400/1,000 features)
- bench::mark reports median elapsed time and memory from 20 repetitions per package per size, aggregated into benchmark_results.csv
- MetaNet completes construction on datasets with fewer than 1,000 features in under 0.2 seconds and under 100 MB of memory
- Observed speed-up and memory ratios approach the reported up-to-100-fold time and up-to-50-fold memory advantages over comparator packages
- The same correlation method and significance filtering are used across all tools, so timing differences reflect implementation efficiency

## Limitations

- Benchmark measured on a specific hardware/OS combination (Apple M2, macOS); absolute times may not transfer to other platforms
- Only synthetic matrices up to 1,000 features were profiled in this task, though MetaNet claims scalability beyond 10,000 features
- MetaNet's efficiency stems from optimized vectorized matrix algorithms for correlation coefficients and p-values, but correlation-based construction 'may overlook complex, non-linear, or indirect associations'

## Evidence

- [abstract] Benchmarking shows MetaNet outperforms existing R packages by up to 100-fold in computation time and reduces memory usage by up to 50-fold.: [evidence span withheld — anchor status: not_found]
- [readme] MetaNet addresses this through optimized vectorized matrix algorithms: [evidence span withheld — anchor status: external_doc]
- [intro] Computational performance is a bottleneck: "computational performance is often a bottleneck—especially for correlation-based network construction on high-dimensional datasets, which may require hours or even days25."
- [abstract] supports highly efficient correlation-based network construction, scalable to datasets with over 10,000 features: "supports highly efficient correlation-based network construction, scalable to datasets with over 10,000 features"
