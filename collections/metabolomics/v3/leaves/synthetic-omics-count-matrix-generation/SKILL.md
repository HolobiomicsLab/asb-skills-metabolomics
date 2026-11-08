---
name: synthetic-omics-count-matrix-generation
description: Use when benchmarking the computational cost of correlation-based network
  construction across R packages (e.g. MetaNet vs WGCNA or NetCoMi) and reproducible,
  identical synthetic count-matrix inputs are needed.
when_to_use_negative:
- The goal is to analyze real biological omics data and interpret network biology,
  not to benchmark software runtime or memory usage.
- Datasets exceed 10,000 features, which is outside the 50–1,000 feature range benchmarked
  here, though MetaNet claims scalability beyond 10,000 features.
- Only one network construction tool is being used — comparative runtime/memory measurement
  requires the matched comparator packages.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: MetaNet
  role: Primary tool under benchmark; its c_net_calculate performs the correlation-based
    network construction being timed
  repo: https://cran.r-project.org/web/packages/MetaNet
- name: R
  role: Execution environment for generating synthetic matrices, running bench::mark,
    and aggregating results
- name: WGCNA
  role: Comparator package; its corAndPvalue function is timed against c_net_calculate
- name: microeco
  role: Comparator package; its trans_network$new constructor is timed against c_net_calculate
- name: ggClusterNet
  role: Comparator package; its corMicro function is timed against c_net_calculate
- name: NetCoMi
  role: Comparator package; its netConstruct function is timed against c_net_calculate
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:c5a5acc826d783b3fd7621b7753cb811bfd5ada7cb775c99a185a3764d652ae2
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Synthetic omics count matrix generation

## Summary

Generates fixed-seed synthetic feature abundance count matrices at five benchmark sizes (50, 100, 200, 400, 1,000 features) so that MetaNet's c_net_calculate and comparator packages (WGCNA, microeco, ggClusterNet, NetCoMi) receive identical inputs when measuring runtime and memory for correlation-based network construction.

## When to use

Use when benchmarking the computational cost of correlation-based network construction across R packages (e.g. MetaNet vs WGCNA or NetCoMi) and reproducible, identical synthetic count-matrix inputs are needed.

## When NOT to use

- The goal is to analyze real biological omics data and interpret network biology, not to benchmark software runtime or memory usage.
- Datasets exceed 10,000 features, which is outside the 50–1,000 feature range benchmarked here, though MetaNet claims scalability beyond 10,000 features.
- Only one network construction tool is being used — comparative runtime/memory measurement requires the matched comparator packages.

## Inputs

- Synthetic feature abundance count matrices generated at five sizes (50, 100, 200, 400, 1,000 features) with a fixed random seed
- R environment with MetaNet installed via devtools::install_github("Asa12138/MetaNet"), its pcutils dependency, and the comparator packages WGCNA, microeco, ggClusterNet, and NetCoMi

## Outputs

- bench::mark timing records (elapsed time and memory allocation per construction call, 20 repetitions)
- benchmark_results.csv containing median computation time and memory per package per dataset size

## How to apply

Generate synthetic feature abundance count matrices at five sizes — 50, 100, 200, 400, and 1,000 features — using a fixed random seed so all packages receive identical inputs. For each matrix, run each tool's correlation-network construction: c_net_calculate(totu, totu2, method, p.adjust.method, threads, ...) for MetaNet, corAndPvalue for WGCNA, trans_network$new for microeco, corMicro for ggClusterNet, and netConstruct for NetCoMi, applying the same correlation method and significance filtering across tools. Time every construction call with bench::mark using 20 repetitions (the reference run was on an Apple M2 macOS system), capturing elapsed time and memory allocation. Aggregate the bench::mark results into a table of median computation time and memory per package per dataset size, then write the summary to benchmark_results.csv. Verify that MetaNet shows the reported advantage — up to 100-fold speed and 50-fold memory reduction, with sub-1,000-feature datasets completing in under 0.2 seconds and under 100 MB.

## Related tools

- **MetaNet** (Primary tool under benchmark; its c_net_calculate performs the correlation-based network construction being timed) — https://cran.r-project.org/web/packages/MetaNet
- **R** (Execution environment for generating synthetic matrices, running bench::mark, and aggregating results)
- **WGCNA** (Comparator package; its corAndPvalue function is timed against c_net_calculate)
- **microeco** (Comparator package; its trans_network$new constructor is timed against c_net_calculate)
- **ggClusterNet** (Comparator package; its corMicro function is timed against c_net_calculate)
- **NetCoMi** (Comparator package; its netConstruct function is timed against c_net_calculate)

## Examples

```
set.seed(123); totu <- t2(otutab[1:70, ]); cor <- c_net_calculate(totu)
```

## Evaluation signals

- All packages receive identical input matrices because a fixed random seed is used for every dataset size.
- The synthetic matrices match the five specified sizes exactly: 50, 100, 200, 400, and 1,000 features.
- benchmark_results.csv exists and contains median computation time and memory per package per dataset size, aggregated from 20 bench::mark repetitions.
- For datasets with fewer than 1,000 features, MetaNet completes in under 0.2 seconds and under 100 MB of memory.
- The observed runtime advantage of MetaNet over comparators is directionally consistent with the reported up-to-100-fold speed and up-to-50-fold memory improvements.

## Limitations

- No version numbers are stated for MetaNet or any compared tools (WGCNA, NetCoMi, ggClusterNet, microeco), so exact reproducibility across versions is not guaranteed.
- Detailed benchmark comparison settings are not fully reported in the available document sections.
- Performance was measured on a single system (Apple M2 macOS), so timings may not generalize to other hardware.
- Correlation-based network construction may overlook complex, non-linear, or indirect associations, and synthetic count matrices may not reflect real omics sparsity or structure.

## Evidence

- [readme] README documents the MetaNet quick-start workflow of c_net_calculate, c_net_build, and c_net_plot used in the benchmark.: [evidence span withheld — anchor status: external_doc]
- [readme] README documents the fixed quick-start pattern of transposing the count table with pcutils::t2 before c_net_calculate.: [evidence span withheld — anchor status: external_doc]
