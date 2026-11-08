---
name: monte-carlo-result-aggregation
description: Use when a simulation study produces per-replicate, per-pathway test
  calls (e.g., NetGSA REHE vs REML power runs) and you need per-method empirical power
  estimated across replicates and summarized as mean +/- SE within defined effect-size
  bins.
when_to_use_negative:
- The data are real observed samples rather than simulated replicates — empirical
  power/type-I error cannot be estimated without known ground-truth dysregulated genes
- Only a single method/run exists with no per-replicate replicate-level calls to aggregate
- The goal is differential expression testing on raw data rather than summarizing
  simulation outputs across bins
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: netgsa
  role: R package providing NetGSA.REHE and REML pathway tests whose replicate-level
    results are aggregated; installed from GitHub
  repo: github.com/mikehellstern/netgsa
- name: NetGSA
  role: The overhauled NetGSA method whose main function returns test statistics and
    pathway-level calls across simulated replicates for power and type-I error estimation
  repo: github.com/mikehellstern/netgsa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:8593787b012a0bfab871f138f28b0167240aa829a1f784cfaa3aeeba84be2735
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NetGSA
  - netgsa
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# monte-carlo-result-aggregation

## Summary

Aggregates replicate-level results from Monte Carlo simulation studies (e.g., empirical power and type-I error across methods like NetGSA.REHE with/without clustering and REML) into summary tables binned by effect condition, written to CSV for downstream comparison.

## When to use

Use when a simulation study produces per-replicate, per-pathway test calls (e.g., NetGSA REHE vs REML power runs) and you need per-method empirical power estimated across replicates and summarized as mean +/- SE within defined effect-size bins.

## When NOT to use

- The data are real observed samples rather than simulated replicates — empirical power/type-I error cannot be estimated without known ground-truth dysregulated genes
- Only a single method/run exists with no per-replicate replicate-level calls to aggregate
- The goal is differential expression testing on raw data rather than summarizing simulation outputs across bins

## Inputs

- Per-replicate, per-pathway test statistics and significance calls from each compared method (NetGSA.REHE with clustering, NetGSA.REHE without clustering, REML)
- Simulation metadata mapping each pathway/gene group to its dysregulation signal condition (0.2, 0.3, 0.4; group sizes None, (0,5], (5,10], >10)
- Predefined bin definitions for aggregation (dysregulation-group bins)

## Outputs

- Aggregated power summary table (power_summary_fig5a.csv) with mean power +/- standard error per method per dysregulation-group bin
- Empirical type-I error estimate for the 'None' (no-signal) bin per method

## How to apply

Run the simulation replicates (e.g., dysregulated-gene signals of 0.2, 0.3, and 0.4 added to gene groups of varying sizes on the centered/scaled breastcancer2012 expression data) and collect test statistics and pathway-level calls for each method and replicate. For each pathway and method, estimate empirical power as the fraction of replicates with a significant call. Bin pathways by dysregulation-group size — None, (0,5], (5,10], >10 — and compute mean power and its standard error within each bin for each method. Write the aggregated table to a CSV (e.g., power_summary_fig5a.csv). Validate correctness by checking the invariant that the 'None' bin has mean power approximately 0 for all methods (type-I error control) and that the effect-carrying bins show the expected method ranking (REHE with clustering > REHE alone or REML), consistent with the paper's claim of no loss of statistical power.

## Related tools

- **netgsa** (R package providing NetGSA.REHE and REML pathway tests whose replicate-level results are aggregated; installed from GitHub) — github.com/mikehellstern/netgsa
- **NetGSA** (The overhauled NetGSA method whose main function returns test statistics and pathway-level calls across simulated replicates for power and type-I error estimation) — github.com/mikehellstern/netgsa

## Examples

```
Rscript aggregate_power.R --methods REHE_clust,REHE,REML --bins None,"(0,5]","(5,10]",">10" --out power_summary_fig5a.csv
```

## Evaluation signals

- Mean power in the 'None' dysregulation-group bin is approximately 0 for all three methods (type-I error control)
- For all dysregulated-gene group bins, mean power for REHE with clustering is higher than REHE without clustering and REML
- Summary table has one mean +/- standard error row per method per bin (None, (0,5], (5,10], >10) and is written to power_summary_fig5a.csv
- Power values fall in [0,1] and standard errors decrease as the number of replicates grows

## Limitations

- The validation only covers the 'None' bin's type-I error; the README notes compatibility checks between adjacency matrices and input data were removed and 'should be added later', so malformed inputs could silently propagate into aggregated statistics
- Results depend on the simulated dysregulation frameworks (signal magnitudes 0.2/0.3/0.4 and group sizes) and may not transfer to other effect designs

## Evidence

- [abstract] Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power: "Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power"
- [readme] Variance component estimation note: [evidence span withheld — anchor status: external_doc]
- [readme] Filtering genes before NetGSA: [evidence span withheld — anchor status: external_doc]
