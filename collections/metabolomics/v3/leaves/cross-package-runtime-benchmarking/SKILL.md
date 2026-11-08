---
name: cross-package-runtime-benchmarking
description: Use when comparing computational efficiency (elapsed time, memory allocation)
  of alternative R packages performing the same network-construction task on identically
  sized, identically seeded inputs, e.g. to substantiate claims like 'up to 100-fold
  faster'.
when_to_use_negative:
- The goal is benchmarking wall-clock speed rather than network quality — topology
  or biological validity of resulting networks is out of scope
- Datasets exceed ~1,000 features per tool but the packages' own documented limits
  differ; scaling beyond the paper's tested sizes (e.g. >10,000 features) was not
  validated here
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaNet
  role: Package under test providing c_net_calculate for correlation-based network
    construction
  repo: github.com/Asa12138/MetaNet
- name: WGCNA
  role: Comparator package; corAndPvalue used as its construction entry point
- name: microeco
  role: Comparator package; trans_network$new used as its construction entry point
- name: ggClusterNet
  role: Comparator package; corMicro used as its construction entry point
- name: NetCoMi
  role: Comparator package; netConstruct used as its construction entry point
- name: R
  role: Execution environment for all benchmark runs and bench::mark timing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:005a730cdde456ddb5e53ba221210a69dd1370222a15ee74d6e5e582d322bf2c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaNet
  - NetCoMi
  - R
  - WGCNA
  - ggClusterNet
  - microeco
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# cross-package-runtime-benchmarking

## Summary

A reproducible runtime/memory benchmarking skill that times equivalent network-construction calls from R packages such as MetaNet, WGCNA, microeco, ggClusterNet, and NetCoMi on identically seeded synthetic count matrices of graded sizes (50–1,000 features), using bench::mark over repeated runs to quantify per-package speed and memory scaling.

## When to use

Use when comparing computational efficiency (elapsed time, memory allocation) of alternative R packages performing the same network-construction task on identically sized, identically seeded inputs, e.g. to substantiate claims like 'up to 100-fold faster'.

## When NOT to use

- The goal is benchmarking wall-clock speed rather than network quality — topology or biological validity of resulting networks is out of scope
- Datasets exceed ~1,000 features per tool but the packages' own documented limits differ; scaling beyond the paper's tested sizes (e.g. >10,000 features) was not validated here

## Inputs

- Synthetic feature-abundance count matrix (features x samples) at each benchmark size: 50, 100, 200, 400, and 1,000 features, generated with a fixed seed
- Installed R packages under test: MetaNet, WGCNA, microeco, ggClusterNet, NetCoMi

## Outputs

- bench::mark results (elapsed time and memory allocation per call)
- Summary table of median computation time and memory per package per dataset size (e.g. benchmark_results.csv)

## How to apply

Install the packages under test (MetaNet via devtools::install_github('Asa12138/MetaNet'), plus WGCNA, microeco, ggClusterNet, NetCoMi) and load them in a single R session so all tools run on the same machine. Generate synthetic feature-abundance count matrices at graded sizes — 50, 100, 200, 400, and 1,000 features — with a fixed random seed so every package receives byte-identical inputs. For each matrix, invoke each tool's own construction entry point — c_net_calculate for MetaNet, corAndPvalue for WGCNA, trans_network$new for microeco, corMicro for ggClusterNet, netConstruct for NetCoMi — with the same correlation method and significance filtering across tools. Time each call with bench::mark over 20 repetitions, capturing median elapsed time and memory allocation. Aggregate the bench::mark output into a per-package, per-size summary table and check that relative speedups match claimed ratios (e.g. MetaNet completing <1,000-feature datasets in under 0.2 s / <100 MB).

## Related tools

- **MetaNet** (Package under test providing c_net_calculate for correlation-based network construction) — github.com/Asa12138/MetaNet
- **WGCNA** (Comparator package; corAndPvalue used as its construction entry point)
- **microeco** (Comparator package; trans_network$new used as its construction entry point)
- **ggClusterNet** (Comparator package; corMicro used as its construction entry point)
- **NetCoMi** (Comparator package; netConstruct used as its construction entry point)
- **R** (Execution environment for all benchmark runs and bench::mark timing)

## Examples

```
Rscript -e 'library(bench); cor <- c_net_calculate(totu); bench::mark(c_net_calculate(totu), corAndPvalue(totu), netConstruct(totu), replications=20)'
```

## Evaluation signals

- All packages receive identical input matrices (same seed, same feature counts) — no leakage of different data per package
- Same correlation method and significance filtering applied uniformly across c_net_calculate, corAndPvalue, trans_network$new, corMicro, and netConstruct
- bench::mark repeated ≥20 times per call; report median (not single-run) elapsed time and memory allocation
- MetaNet's measured times/memory should show a clear advantage consistent with the paper's claims of up to 100-fold speed and 50-fold memory reduction, with <1,000-feature runs finishing under 0.2 s and <100 MB
- Final table covers every package x every dataset size (5 packages x 5 sizes = 25 benchmark cells) with no missing cells

## Limitations

- The article's benchmarking details are not fully reported in the available sections (only headline 100-fold / 50-fold figures), so exact per-cell timings must be reproduced rather than read off
- No version numbers stated for MetaNet or any compared tools, so results are only comparable if the same versions are installed
- Benchmark uses synthetic count matrices up to 1,000 features; behaviour on real high-dimensional data (e.g. >10,000 features) was claimed but not validated in this benchmark setup

## Evidence

- [results] MetaNet outperformed existing R packages by up to 100-fold in computation time and reduced memory usage by up to 50-fold, completing analysis on datasets with fewer than 1,000 features in under 0.2 seconds and under 100 MB of memory.: [evidence span withheld — anchor status: not_found]
- [methods] Time every construction call with bench::mark using 20 repetitions: [evidence span withheld — anchor status: not_found]
- [intro] Computational performance is a bottleneck for correlation-based network construction on high-dimensional datasets, potentially taking hours or days: [evidence span withheld — anchor status: not_found]
