---
name: omics-network-construction-benchmarking
description: Use when evaluating or comparing the computational efficiency of network
  construction tools (e.g. correlation-based co-occurrence builders) on synthetic
  or real omics datasets spanning a range of feature counts, to quantify how runtime
  and memory scale with dimensionality.
when_to_use_negative:
- When the goal is to compare network topology quality or biological validity rather
  than computational scaling — the skill only benchmarks time and memory.
- When the input is already a built network or adjacency matrix and construction cost
  is not the quantity of interest.
- When evaluating construction cost on a single dataset size; the scaling exponent
  requires a range of feature counts.
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: MetaNet
  role: Primary benchmarked tool; correlation-based network construction via `c_net_calculate`
    (Pearson/Spearman with p-value adjustment) and `c_net_build` at a specified r_threshold.
  repo: github.com/Asa12138/MetaNet
- name: pcutils
  role: Companion package providing the synthetic omics test data (`otutab`) used
    for the quick-start benchmark and plotting utilities.
  repo: github.com/Asa12138/pcutils
- name: igraph
  role: Underlying graph engine loaded alongside MetaNet for network representation
    and topological access during the benchmark.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:e2b7cf503fda8dddd843e5fa9810d423301e502dc1778cf20620fb231b6b7710
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaNet
  - R
  - igraph
  - pcutils
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# omics-network-construction-benchmarking

## Summary

Benchmarks the computational performance (runtime and peak memory) of correlation-based omics network construction tools across a series of increasing feature counts, fitting log-log scaling curves against feature number.

## When to use

Use when evaluating or comparing the computational efficiency of network construction tools (e.g. correlation-based co-occurrence builders) on synthetic or real omics datasets spanning a range of feature counts, to quantify how runtime and memory scale with dimensionality.

## When NOT to use

- When the goal is to compare network topology quality or biological validity rather than computational scaling — the skill only benchmarks time and memory.
- When the input is already a built network or adjacency matrix and construction cost is not the quantity of interest.
- When evaluating construction cost on a single dataset size; the scaling exponent requires a range of feature counts.

## Inputs

- Seeded synthetic omics count tables at multiple feature sizes (e.g. 100–5000 features)
- Network construction tool callable with fixed correlation method and threshold parameters
- Benchmark target values from the reference study (e.g. <0.2 s and <100 MB for <1000 features)

## Outputs

- Per-size runtime and peak-memory measurements
- Fitted log-log scaling exponents for time and memory vs. feature number
- benchmark_results.csv summarizing measurements and fitted exponents
- Scaling plot (e.g. scaling_plot.png) showing runtime/memory vs. feature count

## How to apply

Generate seeded synthetic omics count tables spanning a series of feature sizes (e.g. 100, 250, 500, 750, 1000, 2500, 5000) that bracket the scale regime of interest. For each size, run the network construction step with identical settings — in MetaNet's case `c_net_calculate` for Pearson/Spearman correlation with a specified `p.adjust.method`/`p.adjust.mode` — while timing the call with `system.time` and recording peak memory via `gc()` or R memory profiling. Establish the reference targets from the published benchmark: MetaNet completes correlation-based analysis on fewer than 1,000 features in under 0.2 seconds with less than 100 MB of memory, and claims up to 100-fold speedup and 50-fold memory reduction over existing R packages. Fit log-log linear regressions of elapsed time and peak memory against feature number; the expected fitted exponent is approximately 2 (quadratic scaling, as in the paper's Figure 1D/1E). Write the measurements and fitted exponents to a results table (e.g. benchmark_results.csv) and save the scaling plot.

## Related tools

- **MetaNet** (Primary benchmarked tool; correlation-based network construction via `c_net_calculate` (Pearson/Spearman with p-value adjustment) and `c_net_build` at a specified r_threshold.) — github.com/Asa12138/MetaNet
- **pcutils** (Companion package providing the synthetic omics test data (`otutab`) used for the quick-start benchmark and plotting utilities.) — github.com/Asa12138/pcutils
- **igraph** (Underlying graph engine loaded alongside MetaNet for network representation and topological access during the benchmark.)

## Examples

```
Rscript -e 'devtools::install_github("Asa12138/pcutils"); library(pcutils); library(MetaNet); data("otutab", package="pcutils"); cor <- c_net_calculate(t2(otutab[1:70, ])); net <- c_net_build(cor, r_threshold=0.65)'
```

## Evaluation signals

- All runs on datasets with fewer than 1,000 features complete in under 0.2 s with under 100 MB peak memory (paper's stated benchmark).
- Log-log regression of elapsed time and peak memory against feature number yields an exponent of approximately 2 (quadratic scaling).
- Correlation method and thresholding settings are identical across all feature sizes (controlled variable check).
- benchmark_results.csv contains one row per feature size with both runtime and peak-memory columns populated.
- Scaling plot is saved as scaling_plot.png and visually matches the paper's Figure 1D/1E style.

## Limitations

- MetaNet's quadratic scaling applies to correlation-based construction; complex, non-linear, or indirect associations may be overlooked by such networks.
- Benchmark results depend on the correlation method and p-value adjustment settings; using different settings across sizes invalidates the scaling fit.
- Memory-usage and runtime targets (100-fold speedup, 50-fold memory reduction) were measured against 'existing R packages'; exact competitor versions are not stated in the article, so reproducibility of the comparison requires re-running the same competitors.
- MetaNet requires R version 4.0 or higher and operates on operating systems (Windows, macOS, and Linux) that support it.

## Evidence

- [results] Benchmarking showed that MetaNet completed correlation-based analysis on datasets with fewer than 1,000 features in under 0.2 seconds while using less than 100 MB of memory, establishing the runtime and memory targets to reproduce.: [evidence span withheld — anchor status: not_found]
- [readme] MetaNet addresses this through optimized vectorized matrix algorithms for calculating correlation coefficients and corresponding p-values, greatly reducing memory use and runtime (Figure 1D).: [evidence span withheld — anchor status: external_doc]
- [readme] totu <- t2(otutab[1:70, ]): [evidence span withheld — anchor status: external_doc]
