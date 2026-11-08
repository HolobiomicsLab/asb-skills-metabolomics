---
name: statistical-power-estimation
description: Use when evaluating whether a revised pathway enrichment implementation
  (e.g., NetGSA REHE) retains power and type-I error control under simulated dysregulation
  before trusting its pathway-level calls.
when_to_use_negative:
- When the goal is enrichment of a real (non-simulated) expression dataset rather
  than benchmarking method power/type-I error
- When comparing first/second-generation enrichment methods (e.g., SPIA, PRS) that
  require pre-called differentially expressed genes, since the REHE simulation framework
  targets topology-based NetGSA variants
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0605
tools:
- name: netgsa
  role: R package providing NetGSA.REHE (with and without gene clustering into subnetworks)
    and REML used to compute pathway test statistics on each simulated replicate
  repo: github.com/mikehellstern/netgsa
- name: NetGSA
  role: Underlying methodology whose overhauled implementation is benchmarked for
    power and type-I error in the simulation study
  repo: github.com/mikehellstern/netgsa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:0535d52ce69b951af85f8f60a33329e10c1cb3881d56af70c6f64f78debfba04
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NetGSA
  - netgsa
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# statistical-power-estimation

## Summary

Empirically estimate per-pathway statistical power and type-I error for pathway enrichment methods (NetGSA REHE with/without clustering, REML) by simulating dysregulated-gene signals on the breastcancer2012 expression dataset and aggregating power across dysregulation-group bins.

## When to use

Use when evaluating whether a revised pathway enrichment implementation (e.g., NetGSA REHE) retains power and type-I error control under simulated dysregulation before trusting its pathway-level calls.

## When NOT to use

- When the goal is enrichment of a real (non-simulated) expression dataset rather than benchmarking method power/type-I error
- When comparing first/second-generation enrichment methods (e.g., SPIA, PRS) that require pre-called differentially expressed genes, since the REHE simulation framework targets topology-based NetGSA variants

## Inputs

- breastcancer2012 gene expression dataset (centered/scaled to unit variance)
- Pathway/gene adjacency structures (e.g., KEGG-derived networks) for subnetwork clustering
- Simulated dysregulation frameworks (mean signals 0.2, 0.3, 0.4 applied to gene groups of varying sizes)

## Outputs

- power_summary_fig5a.csv — aggregated mean power (+/- standard error) per dysregulation-group bin (None, (0,5], (5,10], >10) for each method
- Per-pathway empirical power estimates for NetGSA.REHE with clustering, NetGSA.REHE without clustering, and REML

## How to apply

Load the breastcancer2012 gene expression dataset and center/scale it to unit variance. Simulate dysregulated-gene signals under three dysregulation frameworks by adding mean signals of 0.2, 0.3, and 0.4 to groups of dysregulated genes of varying sizes. For each simulated replicate and pathway, run NetGSA.REHE with clustering of genes into subnetworks, NetGSA.REHE without clustering, and REML to obtain test statistics and pathway-level calls. Estimate per-pathway empirical power for each method across replicates, then aggregate mean power (+/- standard error) within dysregulation-group bins (None, (0,5], (5,10], >10). Write the aggregated table to power_summary_fig5a.csv.

## Related tools

- **netgsa** (R package providing NetGSA.REHE (with and without gene clustering into subnetworks) and REML used to compute pathway test statistics on each simulated replicate) — github.com/mikehellstern/netgsa
- **NetGSA** (Underlying methodology whose overhauled implementation is benchmarked for power and type-I error in the simulation study) — github.com/mikehellstern/netgsa

## Examples

```
library(netgsa); res <- NetGSA.REHE(x, pathways, clustering=TRUE); power_summary_fig5a <- aggregate(mean_power ~ bin + method, results, mean)
```

## Evaluation signals

- Mean power in the 'None' dysregulation group is approximately 0 for all three methods, demonstrating type-I error control
- For all non-'None' dysregulated gene groups, mean power using REHE with clustering exceeds REHE alone and REML
- Power increases monotonically with dysregulation signal magnitude (0.2, 0.3, 0.4) and group size bins ((0,5], (5,10], >10)
- Output table power_summary_fig5a.csv contains mean power plus standard errors for each method x bin combination
- Simulated expression matrix has unit variance per gene after centering/scaling

## Limitations

- Type-I error/power conclusions rest on the specific simulated dysregulation frameworks and signal magnitudes (0.2-0.4), not real dysregulation
- README notes the compatibility check between adjacency matrices and input data was removed for block-diagonal structures and 'should be added later', so inputs must be validated manually
- Variables in the input data matrix should be filtered to keep only those belonging to at least one tested pathway before running NetGSA

## Evidence

- [results] Power analysis via simulated dysregulation frameworks: [evidence span withheld — anchor status: not_found]
- [other] Power and type-I error validation: [evidence span withheld — anchor status: not_found]
- [other] Simulated signal magnitudes: [evidence span withheld — anchor status: not_found]
- [abstract] Overhauled method retains power: "Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power"
- [readme] Pre-run variable filtering: [evidence span withheld — anchor status: external_doc]
