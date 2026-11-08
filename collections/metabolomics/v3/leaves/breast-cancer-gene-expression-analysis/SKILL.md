---
name: breast-cancer-gene-expression-analysis
description: Use when running a power comparison of topology-based pathway enrichment
  methods on simulated breast cancer expression data, with pathways grouped by dysregulated-gene
  count and FDR alpha=0.05 detection.
when_to_use_negative:
- The goal is single-dataset differential pathway testing rather than a power comparison
  of competing topology-based methods across simulated dysregulation levels
- Input pathways lack topology/adjacency information — methods such as SPIA, PRS,
  topologyGSA, or Pathway-Express have specific input requirements and DEGraph/CAMERA
  alternatives may not apply
- Genes in the input data matrix do not belong to at least any tested pathway and
  cannot be filtered to pathway members before running NetGSA
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: netgsa
  role: R package implementing NetGSA with REHE variance-component estimation and
    clustering, used to run the benchmarked method and pathway enrichment analysis
  repo: github.com/mikehellstern/netgsa
- name: NetGSA
  role: Network-based Gene Set Analysis methodology whose test statistics and FDR-thresholded
    p-values are the basis of the power benchmark
  repo: github.com/mikehellstern/netgsa
- name: Pathway-Express
  role: Competing topology-based pathway enrichment method run in its no-cutoff configuration
    for the power comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:5f26ad68e4dcb64eb5e39c8634fb9f722fdb972bbcc02778d46e0cfa740756ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NetGSA
  - Pathway-Express
  - netgsa
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# breast-cancer-gene-expression-analysis

## Summary

Benchmarking skill for comparing the statistical power of topology-based pathway enrichment methods (NetGSA.REHE.Cl vs DEGraph, CAMERA, Pathway-Express) on simulated breast cancer gene-expression datasets, grouping pathways by number of dysregulated genes.

## When to use

Use when running a power comparison of topology-based pathway enrichment methods on simulated breast cancer expression data, with pathways grouped by dysregulated-gene count and FDR alpha=0.05 detection.

## When NOT to use

- The goal is single-dataset differential pathway testing rather than a power comparison of competing topology-based methods across simulated dysregulation levels
- Input pathways lack topology/adjacency information — methods such as SPIA, PRS, topologyGSA, or Pathway-Express have specific input requirements and DEGraph/CAMERA alternatives may not apply
- Genes in the input data matrix do not belong to at least any tested pathway and cannot be filtered to pathway members before running NetGSA

## Inputs

- Breast cancer gene-expression dataset
- Simulated datasets varying the number of dysregulated genes per pathway group
- Pathway/gene-set definitions (e.g. KEGG pathways) and gene-gene interaction (adjacency) information
- FDR significance threshold alpha=0.05

## Outputs

- fig5b_power_table.csv: method-by-group power table (methods as rows, dysregulated-gene-count groups as columns) with standard errors
- Empirical power estimates per method per dysregulated-gene group across simulation replicates

## How to apply

Load the breast cancer dataset and construct simulation frameworks that vary the number of dysregulated genes per pathway group. For each simulated dataset, run NetGSA.REHE.Cl (the netgsa R package's NetGSA methodology with REHE variance-component estimation and clustering) and the competing topology-based methods DEGraph, CAMERA, and Pathway-Express in their no-cutoff configurations. Classify a pathway as detected when its p-value passes the FDR threshold alpha=0.05, and compute empirical power as the fraction of true dysregulated pathways recovered within each dysregulated-gene-count group. Aggregate to mean power per method per group with standard errors across simulation replicates, and write the method-by-group power table to fig5b_power_table.csv. Rationale: the paper shows NetGSA.REHE.Cl is competitive with the top method in all groups, while DEGraph is stronger at low dysregulation and CAMERA/Pathway-Express (no cutoff) are stronger at high dysregulation.

## Related tools

- **netgsa** (R package implementing NetGSA with REHE variance-component estimation and clustering, used to run the benchmarked method and pathway enrichment analysis) — github.com/mikehellstern/netgsa
- **NetGSA** (Network-based Gene Set Analysis methodology whose test statistics and FDR-thresholded p-values are the basis of the power benchmark) — github.com/mikehellstern/netgsa
- **Pathway-Express** (Competing topology-based pathway enrichment method run in its no-cutoff configuration for the power comparison)

## Examples

```
devtools::install_github("mikehellstern/netgsa", build_vignettes=T); results <- NetGSA(x, pathways, group, adjacency_list); pvals <- results$pvals  # classify detected at FDR alpha=0.05
```

## Evaluation signals

- The fig5b_power_table.csv table reproduces the Fig 5B pattern: NetGSA.REHE.Cl is competitive with the top-performing method in all dysregulated-gene groups
- DEGraph shows relatively stronger power at low dysregulation levels, and CAMERA/Pathway-Express (no cutoff) show stronger power at high dysregulation levels
- Empirical power per method per group equals the fraction of true dysregulated pathways whose p-values pass FDR alpha=0.05
- Standard errors are computed across simulation replicates and included alongside mean power in the table
- Schema check: methods as rows, dysregulated-gene-count groups as columns, with standard error columns present

## Limitations

- There is no consistent best choice among the many topology-based methods proposed for a given problem; relative power depends on the dysregulation level
- The NetGSA implementation removed the compatibility check between adjacency matrices and input data for block-diagonal structures, which should be re-added later
- Before running NetGSA, variables in the input data matrix must be filtered to keep only genes belonging to at least one tested pathway
- The README raises open design questions (e.g. whether to assemble the entire adjacency matrix vs repeatedly subset variables; whether the beta output is needed), so implementation choices may affect results

## Evidence

- [results] In the breast cancer simulation comparison, the new netgsa software (NetGSA.REHE.Cl) performs favorably in statistical power and is competitive with the top-performing method across all dysregulated gene groups.: [evidence span withheld — anchor status: not_found]
- [abstract] The overhauled NetGSA performs pathway enrichment analysis for thousands of genes in minutes on a personal computer without sacrificing statistical power: "Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power"
- [readme] Variance component estimation: Residuals are needed to estimate the variance components. This is done directly without evaluating the fixed effect coefficients.: [evidence span withheld — anchor status: external_doc]
- [intro] Many topology-based methods have been proposed in the literature, but there is no consistent best choice for a given problem.: "Many topology-based methods have been proposed in the literature, but there is no consistent best choice for a given problem."
