---
name: multi-tool-performance-comparison
description: Use when comparing computational efficiency (runtime, memory) of network-construction
  implementations across R packages on identical inputs, or when validating performance
  claims for a new package against established baselines.
when_to_use_negative:
- The goal is accuracy or topological correctness of constructed networks rather than
  computational efficiency — this skill only measures time and memory
- Datasets exceed the benchmarked scale (>1,000 features in the source protocol);
  the article's headline numbers (<0.2 s, <100 MB) are calibrated to the 50–1,000-feature
  range, though MetaNet claims scalability past 10,000 features
- Comparing tools whose correlation methods or significance thresholds differ, since
  the benchmark's validity depends on identical inputs and parameters across packages
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_1317
tools:
- name: MetaNet
  role: Candidate tool whose c_net_calculate correlation-based network construction
    is being benchmarked
  repo: https://github.com/Asa12138/MetaNet
- name: R
  role: Language environment hosting all benchmarked packages and the bench::mark
    timing harness
- name: WGCNA
  role: Baseline comparator run via corAndPvalue on the same synthetic matrices
- name: microeco
  role: Baseline comparator run via trans_network$new
- name: ggClusterNet
  role: Baseline comparator run via corMicro
- name: NetCoMi
  role: Baseline comparator run via netConstruct
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:315b5889d6db4239d000d15ad739769dfa64e8127da90db89c5fa2004f364a01
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
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_1317
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# multi-tool-performance-comparison

## Summary

Benchmarks MetaNet's c_net_calculate correlation-network construction against WGCNA, microeco, ggClusterNet, and NetCoMi on identical synthetic omics matrices of increasing size (50–1,000 features), quantifying elapsed time and memory with bench::mark to verify reported up-to-100-fold speed and 50-fold memory advantages.

## When to use

Use when comparing computational efficiency (runtime, memory) of network-construction implementations across R packages on identical inputs, or when validating performance claims for a new package against established baselines.

## When NOT to use

- The goal is accuracy or topological correctness of constructed networks rather than computational efficiency — this skill only measures time and memory
- Datasets exceed the benchmarked scale (>1,000 features in the source protocol); the article's headline numbers (<0.2 s, <100 MB) are calibrated to the 50–1,000-feature range, though MetaNet claims scalability past 10,000 features
- Comparing tools whose correlation methods or significance thresholds differ, since the benchmark's validity depends on identical inputs and parameters across packages

## Inputs

- Synthetic feature abundance count matrices at 50, 100, 200, 400, and 1,000 features (identical across packages via fixed seed)
- Correlation method and p-value adjustment settings applied uniformly to all tools

## Outputs

- benchmark_results.csv with median computation time and memory per package per dataset size
- bench::mark output capturing elapsed time and memory allocation over 20 repetitions

## How to apply

Install MetaNet from GitHub via devtools::install_github("Asa12138/MetaNet") with its pcutils dependency, and load comparators WGCNA (corAndPvalue), microeco (trans_network$new), ggClusterNet (corMicro), and NetCoMi (netConstruct). Generate synthetic feature abundance count matrices at five sizes — 50, 100, 200, 400, and 1,000 features — with a fixed random seed so every tool receives identical inputs. Run each package's network-construction entry point using the same correlation method and significance filtering, timing each call with bench::mark using 20 repetitions on a consistent system (Apple M2 macOS in the source study). Aggregate median elapsed time and memory allocation per package per dataset size into a summary table (benchmark_results.csv), then check that the candidate tool shows the expected order-of-magnitude advantage — the study's invariant being under 0.2 s and under 100 MB for fewer than 1,000 features.

## Related tools

- **MetaNet** (Candidate tool whose c_net_calculate correlation-based network construction is being benchmarked) — https://github.com/Asa12138/MetaNet
- **R** (Language environment hosting all benchmarked packages and the bench::mark timing harness)
- **WGCNA** (Baseline comparator run via corAndPvalue on the same synthetic matrices)
- **microeco** (Baseline comparator run via trans_network$new)
- **ggClusterNet** (Baseline comparator run via corMicro)
- **NetCoMi** (Baseline comparator run via netConstruct)

## Examples

```
library(bench); res <- mark(metanet = c_net_calculate(totu), wgcna = WGCNA::corAndPvalue(totu), iterations = 20)
```

## Evaluation signals

- All five package-size combinations produce a bench::mark result with the same fixed-seed input matrix, confirming a controlled comparison
- Median time and memory aggregated over 20 repetitions per package per size are recorded in benchmark_results.csv
- MetaNet completes construction on datasets with fewer than 1,000 features in under 0.2 seconds and under 100 MB of memory
- The runtime/memory ratio between MetaNet and each comparator falls in the direction of the reported up-to-100-fold time and up-to-50-fold memory improvements
- Identical correlation method and significance filtering are verifiably applied across corAndPvalue, trans_network$new, corMicro, netConstruct, and c_net_calculate

## Limitations

- Benchmark figures are platform-specific (Apple M2 macOS in the source workflow) and may not transfer to other hardware
- The benchmark protocol uses synthetic count matrices up to 1,000 features, not the >10,000-feature datasets the article claims scalability for
- Comparison details are not fully reported in the article's discussion section ('Benchmark comparison details not reported in this section'), so exact per-tool configurations must be reconstructed from the workflow card

## Evidence

- [abstract] Benchmarking shows MetaNet outperforms existing R packages by up to 100-fold in computation time and reduces memory usage by up to 50-fold.: [evidence span withheld — anchor status: not_found]
