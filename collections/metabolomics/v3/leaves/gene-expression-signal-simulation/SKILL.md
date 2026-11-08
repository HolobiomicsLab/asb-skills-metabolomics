---
name: gene-expression-signal-simulation
description: 'Use when benchmarking pathway enrichment methods on simulated signals:
  you have a real expression matrix, need ground-truth dysregulated gene groups of
  varying sizes, and must measure empirical power and type-I error across replicates.'
when_to_use_negative:
- The goal is descriptive differential-expression analysis rather than method benchmarking
  — no simulation of signals or power estimation is needed
- Expression data already contain known phenotype labels and no ground-truth dysregulation
  simulation is required for validation
- The method under test requires differentially expressed genes as input (e.g., SPIA,
  PRS), which the paper notes 'may or may not be detected' — a different evaluation
  design applies
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: netgsa
  role: R package providing NetGSA.REHE (with and without clustering of genes into
    subnetworks) and REML implementations used to compute pathway test statistics
    on the simulated data
  repo: github.com/mikehellstern/netgsa
- name: NetGSA
  role: The topology-based pathway enrichment method whose overhauled implementation
    is being evaluated for statistical power and type-I error on the simulated dysregulation
    frameworks
  repo: github.com/mikehellstern/netgsa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:6f2831eaa179eeda3bb544e8dc7057a7d8197393495c4133fe4d907f6fdf800e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NetGSA
  - netgsa
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# gene-expression-signal-simulation

## Summary

Simulates dysregulated-gene signals onto a real gene expression matrix (e.g., breastcancer2012 centered/scaled to unit variance) by adding known mean signals (0.2, 0.3, 0.4) to defined groups of dysregulated genes, enabling empirical power and type-I error evaluation of pathway enrichment methods such as NetGSA.REHE (with and without clustering) and REML.

## When to use

Use when benchmarking pathway enrichment methods on simulated signals: you have a real expression matrix, need ground-truth dysregulated gene groups of varying sizes, and must measure empirical power and type-I error across replicates.

## When NOT to use

- The goal is descriptive differential-expression analysis rather than method benchmarking — no simulation of signals or power estimation is needed
- Expression data already contain known phenotype labels and no ground-truth dysregulation simulation is required for validation
- The method under test requires differentially expressed genes as input (e.g., SPIA, PRS), which the paper notes 'may or may not be detected' — a different evaluation design applies

## Inputs

- Gene expression matrix (e.g., breastcancer2012), preprocessed to unit variance
- Pathway/gene-set definitions (e.g., KEGG, Reactome, PantherDB curated networks)
- Gene-gene interaction adjacency matrices for the tested groups
- Simulation settings: dysregulated gene group sizes, mean signal magnitudes (0.2, 0.3, 0.4), number of replicates

## Outputs

- Per-pathway empirical power estimates for NetGSA.REHE (with clustering), NetGSA.REHE (without clustering), and REML
- Aggregated mean power (+/- SE) table binned by dysregulation-group size (None, (0,5], (5,10], >10), e.g., power_summary_fig5a.csv
- Type-I error estimate from the 'None' dysregulation group

## How to apply

Load a real gene expression dataset (the comparative study uses breastcancer2012) and center/scale it to unit variance. Under three dysregulation frameworks, add mean signals of 0.2, 0.3, and 0.4 to groups of dysregulated genes of varying sizes while leaving a 'None' group undisturbed as a negative control for type-I error. For each simulated replicate and pathway, run NetGSA.REHE with clustering of genes into subnetworks, NetGSA.REHE without clustering, and REML to obtain test statistics and pathway-level calls. Estimate per-pathway empirical power per method across replicates, then aggregate mean power (+/- standard error) within dysregulation-group bins (None, (0,5], (5,10], >10) and write the aggregated table (e.g., power_summary_fig5a.csv). The rationale is to verify that the overhauled netgsa software preserves statistical power while its computational speedups (REHE variance component estimation, efficient contrast vectors) remain valid.

## Related tools

- **netgsa** (R package providing NetGSA.REHE (with and without clustering of genes into subnetworks) and REML implementations used to compute pathway test statistics on the simulated data) — github.com/mikehellstern/netgsa
- **NetGSA** (The topology-based pathway enrichment method whose overhauled implementation is being evaluated for statistical power and type-I error on the simulated dysregulation frameworks) — github.com/mikehellstern/netgsa

## Examples

```
devtools::install_github("mikehellstern/netgsa", build_vignettes=T)
```

## Evaluation signals

- Mean power in the 'None' dysregulation group must be approximately 0 for all three methods, demonstrating type-I error control
- For all non-'None' dysregulated gene groups, mean power using REHE with clustering should be higher than with REHE alone or REML
- Aggregated table (power_summary_fig5a.csv) must contain one mean power (+/- SE) row per method per bin (None, (0,5], (5,10], >10)
- Mean power should increase with signal magnitude (0.2 < 0.3 < 0.4) within each dysregulation-group bin
- Input expression matrix must be confirmed centered/scaled to unit variance before signal injection

## Limitations

- The README notes the removed compatibility check between adjacency matrices and input data ('I removed the check on variable compatibility between the adjacency matrices and the input data, but this should be added later'), so users must manually ensure block-diagonal structure consistency
- Variables in the input data matrix should be filtered to keep only those belonging to at least one tested pathway before running NetGSA
- Fixed effect coefficient beta handling is flagged as an open question in the README ('do we need it? Is there a better way of estimating beta given the block diagonal structure of D?')
- Missing signals note: Discussion section content not present, so caveats from the discussion could not be traced

## Evidence

- [other] power analysis and type-I error validation: [evidence span withheld — anchor status: not_found]
- [abstract] no power sacrificed claim: "Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power"
- [readme] package purpose: "This package carries out Network-based Gene Set Analysis by incorporating external information about interactions among genes"
- [readme] simulation input requirement: [evidence span withheld — anchor status: external_doc]
- [intro] overhauled implementation: "we have completely overhauled the netgsa R package, an implementation of the NetGSA methodology"
