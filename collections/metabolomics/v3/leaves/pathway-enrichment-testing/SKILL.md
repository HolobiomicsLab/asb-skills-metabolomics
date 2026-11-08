---
name: pathway-enrichment-testing
description: Use when testing whether entire pathways are enriched/dysregulated from
  gene expression data by incorporating pathway topology (gene-gene interactions),
  or when benchmarking enrichment methods' statistical power and type-I error in simulations.
when_to_use_negative:
- Genes are treated as independent (over-representation/gene-set methods without topology)
  and pathway structure should not be modeled
- Only a list of pre-identified differentially expressed genes is available and topology-based
  approaches like SPIA/PRS that require detected DE genes are being considered instead
- Input variables include genes that belong to no tested pathway — the README says
  to filter the data matrix to keep only genes in at least one tested pathway before
  running NetGSA
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: netgsa
  role: R package implementing the overhauled NetGSA methodology; provides NetGSA.REHE
    (with/without clustering) and REML estimation for pathway-level test statistics
  repo: github.com/mikehellstern/netgsa
- name: NetGSA
  role: 'The topology-based statistical method being applied: network-based gene set
    analysis with variance-component (REHE/REML) estimation and pathway-level testing'
  repo: github.com/mikehellstern/netgsa
- name: Cytoscape
  role: Interactive and intuitive network visualization of enriched pathways produced
    by the netgsa software
- name: DEGraph
  role: Competing topology-based method run in no-cutoff configuration for power benchmarking
    against NetGSA.REHE.Cl
- name: CAMERA
  role: Competing method run in no-cutoff configuration for power benchmarking against
    NetGSA.REHE.Cl
- name: Pathway-Express
  role: Competing topology-based method (PE.noCut) run without cut-offs for power
    benchmarking against NetGSA.REHE.Cl
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:e8a559df8cd82ddacd191012aa2901a1f0ff5985be597619d6b85312d5b0c23d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMERA
  - Cytoscape
  - DEGraph
  - NetGSA
  - Pathway-Express
  - netgsa
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# pathway-enrichment-testing

## Summary

Topology-based pathway enrichment testing using the overhauled NetGSA/netgsa R package, which evaluates gene-gene interaction structure within pathways (KEGG, Reactome, PantherDB) via variance-component estimation with REHE, with optional clustering of genes into subnetworks, and benchmarking power against DEGraph, CAMERA, Pathway-Express, SPIA, PRS, and topologyGSA.

## When to use

Use when testing whether entire pathways are enriched/dysregulated from gene expression data by incorporating pathway topology (gene-gene interactions), or when benchmarking enrichment methods' statistical power and type-I error in simulations.

## When NOT to use

- Genes are treated as independent (over-representation/gene-set methods without topology) and pathway structure should not be modeled
- Only a list of pre-identified differentially expressed genes is available and topology-based approaches like SPIA/PRS that require detected DE genes are being considered instead
- Input variables include genes that belong to no tested pathway — the README says to filter the data matrix to keep only genes in at least one tested pathway before running NetGSA

## Inputs

- Gene expression data matrix (e.g. breastcancer2012), centered and scaled to unit variance
- Adjacency matrices A (list across tested groups, each a list of smaller adjacency matrices) encoding gene-gene interactions from KEGG/Reactome/PantherDB
- Optional: simulated dysregulated-gene signals (mean shifts 0.2/0.3/0.4 added to dysregulated gene groups of varying sizes)
- Competing method inputs for benchmarking: no-cutoff configurations of DEGraph, CAMERA, Pathway-Express

## Outputs

- Per-pathway test statistics and p-values with pathway-level enrichment calls at FDR alpha = 0.05
- Empirical power and type-I error tables aggregated by dysregulated-gene-count group with standard errors (e.g. power_summary_fig5a.csv, fig5b_power_table.csv)
- Interactive network visualization via Cytoscape

## How to apply

Load a gene expression dataset (e.g. breastcancer2012) and center/scale it to unit variance, filtering variables to those belonging to at least one tested pathway as the README instructs. Curate gene-gene interaction information directly from external databases (KEGG, Reactome, PantherDB) rather than requiring expert-built pathways, and supply adjacency matrices A as a list across tested groups (each possibly a list of smaller block matrices). Run NetGSA.REHE, optionally with clustering of genes into subnetworks (NetGSA.REHE.Cl), calling pathways at FDR alpha = 0.05; REHE estimates variance components from residuals directly without evaluating fixed-effect coefficients, and contrasts reuse products of contrast vectors via lapply to compute degrees of freedom and test statistics. For power assessment, simulate dysregulated-gene signals by adding mean signals of 0.2, 0.3, and 0.4 to groups of dysregulated genes of varying sizes, replicate the test across simulation runs, and aggregate empirical power per pathway into bins (None, (0,5], (5,10], >10) with standard errors.

## Related tools

- **netgsa** (R package implementing the overhauled NetGSA methodology; provides NetGSA.REHE (with/without clustering) and REML estimation for pathway-level test statistics) — github.com/mikehellstern/netgsa
- **NetGSA** (The topology-based statistical method being applied: network-based gene set analysis with variance-component (REHE/REML) estimation and pathway-level testing) — github.com/mikehellstern/netgsa
- **Cytoscape** (Interactive and intuitive network visualization of enriched pathways produced by the netgsa software)
- **DEGraph** (Competing topology-based method run in no-cutoff configuration for power benchmarking against NetGSA.REHE.Cl)
- **CAMERA** (Competing method run in no-cutoff configuration for power benchmarking against NetGSA.REHE.Cl)
- **Pathway-Express** (Competing topology-based method (PE.noCut) run without cut-offs for power benchmarking against NetGSA.REHE.Cl)

## Examples

```
library(devtools); devtools::install_github("mikehellstern/netgsa", build_vignettes=T)
```

## Evaluation signals

- Type-I error control: mean empirical power in the 'None' dysregulation group must be approximately 0 for all three methods (NetGSA.REHE with clustering, NetGSA.REHE without clustering, REML)
- Power ordering: for all non-None dysregulated gene groups, mean power with REHE + clustering should exceed REHE alone and REML
- Benchmark pattern (Fig 5B): NetGSA.REHE.Cl competitive with the top-performing method in all groups; DEGraph stronger at low dysregulation; CAMERA/Pathway-Express (no cutoff) stronger at high dysregulation
- Aggregate tables use methods as rows and dysregulated-gene-count groups (None, (0,5], (5,10], >10) as columns, each cell with standard errors across simulation replicates
- Runtime check: pathway enrichment analysis for thousands of genes should complete in minutes on a personal computer

## Limitations

- The README notes the compatibility check between adjacency matrices and input data was removed and should be added later; do not assume block-diagonal adjacency structure across groups is enforced
- Whether the fixed effect coefficient beta output from NetGSA is needed, and how best to estimate beta given block-diagonal D, is flagged as open in the README
- Older topology-based tools differ in input requirements and statistical power; SPIA and PRS require differentially expressed genes which may or may not be detected

## Evidence

- [abstract] Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power: "Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power"
- [intro] computationally efficient methods, such as SPIA [6] and PRS [7] require differentially expressed genes which may or may not be detected: "computationally efficient methods, such as SPIA [6] and PRS [7] require differentially expressed genes which may or may not be detected"
- [readme] This package carries out Network-based Gene Set Analysis by incorporating external information about interactions among genes, as well as novel interactions learned from data.: "This package carries out Network-based Gene Set Analysis by incorporating external information about interactions among genes, as well as novel interactions learned from data."
