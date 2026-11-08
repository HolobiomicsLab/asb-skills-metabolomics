---
name: synthetic-omics-table-generation
description: Use when benchmarking MetaNet (or a comparable correlation-network tool)
  across a range of feature counts and you need controlled synthetic count tables
  spanning below and above the 1,000-feature threshold rather than real data.
when_to_use_negative:
- The goal is to reproduce a published case-study network on real data (e.g. the longitudinal
  airborne microbiome or the >40,000-feature exposome–transcriptome integrative network)
  rather than to measure computational scaling.
- You need biologically meaningful co-occurrence structure or taxonomy/annotation
  attributes — synthetic tables carry no real species, pathway, or module signal.
- The analysis target is network visualization or topological comparison rather than
  benchmarking runtime and memory, in which case a real or built-in dataset (e.g.
  otutab from pcutils) is more appropriate.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaNet
  role: Consumes the synthetic tables via c_net_calculate()/c_net_build() for the
    benchmarked correlation-based network construction
  repo: https://github.com/Asa12138/MetaNet
- name: R
  role: Hosts the seeded random data generation, timing with system.time(), memory
    profiling via gc(), and log-log regression fitting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:e5460901bc149dac2011d44d40a1d99b54b69bc845caddd85b97c9a9a4845925
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Asa12138/MetaNet
  tools:
  - MetaNet
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Synthetic Omics Table Generation

## Summary

Generates seeded synthetic omics count tables at increasing feature sizes so that MetaNet's correlation-based network construction and its runtime/memory scaling curves can be benchmarked reproducibly.

## When to use

Use when benchmarking MetaNet (or a comparable correlation-network tool) across a range of feature counts and you need controlled synthetic count tables spanning below and above the 1,000-feature threshold rather than real data.

## When NOT to use

- The goal is to reproduce a published case-study network on real data (e.g. the longitudinal airborne microbiome or the >40,000-feature exposome–transcriptome integrative network) rather than to measure computational scaling.
- You need biologically meaningful co-occurrence structure or taxonomy/annotation attributes — synthetic tables carry no real species, pathway, or module signal.
- The analysis target is network visualization or topological comparison rather than benchmarking runtime and memory, in which case a real or built-in dataset (e.g. otutab from pcutils) is more appropriate.

## Inputs

- Seed value for reproducible random data generation
- List of target feature sizes (e.g. 100, 250, 500, 750, 1000, 2500, 5000) spanning below and above 1,000 features
- Correlation method and thresholding settings (Pearson/Spearman, r_threshold, p.adjust.method/p.adjust.mode) held constant across sizes

## Outputs

- Synthetic omics count tables (e.g. totu, totu2) with a fixed number of samples and the specified feature count
- Per-size timing and peak-memory measurements (benchmark_results.csv)
- Fitted log-log scaling exponent for runtime and memory (≈2, quadratic) and a saved scaling plot (scaling_plot.png)

## How to apply

Generate seeded synthetic omics count tables (e.g. totu, totu2) at a ladder of increasing feature sizes spanning below and above 1,000 features — such as 100, 250, 500, 750, 1000, 2500, 5000 — keeping the sample count and data-generation parameters fixed so that only feature number varies. Feed each table to c_net_calculate() (Pearson/Spearman correlation with p-value adjustment via p.adjust.method/p.adjust.mode) and c_net_build(), timing each run with system.time() and recording peak memory via gc()/R memory profiling, with correlation method and thresholding held identical across sizes. Because feature number is the only manipulated variable, the elapsed time and memory measurements trace a clean scaling curve: the article targets <0.2 s and <100 MB for datasets with fewer than 1,000 features and expects log-log regression of time and memory against feature number to fit an exponent of approximately 2 (quadratic scaling, Figure 1D/1E style). Use a fixed random seed so the benchmark is reproducible and so re-runs reflect algorithmic performance rather than data variation.

## Related tools

- **MetaNet** (Consumes the synthetic tables via c_net_calculate()/c_net_build() for the benchmarked correlation-based network construction) — https://github.com/Asa12138/MetaNet
- **R** (Hosts the seeded random data generation, timing with system.time(), memory profiling via gc(), and log-log regression fitting)

## Examples

```
Rscript -e "set.seed(42); totu <- matrix(rpois(1000*18, 5), nrow=1000); system.time(cor <- c_net_calculate(t(totu)))"
```

## Evaluation signals

- All runs on datasets with fewer than 1,000 features complete in under 0.2 s with under 100 MB of peak memory
- Log-log linear regression of elapsed time and peak memory against feature number yields an exponent of approximately 2 (quadratic scaling, consistent with Figure 1D/1E)
- Identical correlation method and thresholding settings are used across all sizes; only feature count varies
- Fixed random seed means re-running the generator reproduces byte-identical tables, so timing differences reflect the algorithm, not data variation
- benchmark_results.csv contains one timing and memory row per feature size, and scaling_plot.png shows the fitted curve

## Limitations

- Synthetic tables carry no real biological signal, so results speak to computational scaling only — not to network topology, module detection quality, or biological interpretation
- Feature sizes above 10,000 features are described by the article as the scalable regime but are not required by this skill's target (<1,000 features), so very large sizes may need additional memory-profiling care
- Timing is sensitive to the R version and machine; the article does not report a reference environment, so absolute runtimes should be compared against the relative scaling exponent rather than against wall-clock expectations from other hardware
- p-value adjustment mode (p.adjust.method/p.adjust.mode) must be kept identical across sizes or the comparison becomes confounded

## Evidence

- [readme] Grounds the expected quadratic scaling exponent that the log-log regression should reproduce.: [evidence span withheld — anchor status: external_doc]
- [readme] Provides the concrete R functions (c_net_calculate, c_net_build) that consume the synthetic tables in the benchmark loop.: [evidence span withheld — anchor status: external_doc]
