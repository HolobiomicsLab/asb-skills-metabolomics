---
name: variance-component-estimation-reml
description: Use when running NetGSA pathway enrichment where exact REML variance
  component estimation on the entire weighted, directed pathway network is required,
  or when benchmarking REHE/clustered-REHE runtime and power against the REML gold
  standard.
when_to_use_negative:
- When computational speed is the priority and the REHE approximation suffices — REML
  on the entire network is several times slower than REHE and much slower than clustered
  REHE
- When the input expression matrix contains genes not belonging to any tested pathway
  — variables must first be filtered to pathway members before running NetGSA
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: netgsa
  role: R package providing NetGSA.REML.All, NetGSA.REHE, NetGSA.REHE.Cl, and prepareAdjMat
    for network estimation and pathway enrichment
  repo: https://github.com/mikehellstern/netgsa
- name: knitr
  role: Vignette/rmarkdown engine used to document and reproduce the netgsa workflow
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:ee1f117d49baed56f63428167d0162439c6ec6658f1c1722b2be77a7ecb4ff44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikehellstern/netgsa
  tools:
  - knitr
  - netgsa
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# variance-component-estimation-reml

## Summary

Variance component estimation for NetGSA topology-based pathway enrichment using REML (NetGSA.REML.All) on the full pathway network, serving as the reference (slow but full-network) estimator against which the REHE approximation is benchmarked for power, type-I error, and runtime.

## When to use

Use when running NetGSA pathway enrichment where exact REML variance component estimation on the entire weighted, directed pathway network is required, or when benchmarking REHE/clustered-REHE runtime and power against the REML gold standard.

## When NOT to use

- When computational speed is the priority and the REHE approximation suffices — REML on the entire network is several times slower than REHE and much slower than clustered REHE
- When the input expression matrix contains genes not belonging to any tested pathway — variables must first be filtered to pathway members before running NetGSA

## Inputs

- Gene expression matrix (e.g., breastcancer2012 or TCGA prostate cancer, GEO/TCGA accession from ref [24]), centered/scaled to unit variance
- Curated pathway interaction data (e.g., KEGG, Reactome, PantherDB) for building adjacency matrices
- List of adjacency matrices per pathway per group from prepareAdjMat

## Outputs

- Pathway-level test statistics and enrichment calls from NetGSA.REML.All
- Per-pathway empirical power and type-I error estimates across simulated replicates
- Aggregated mean power (+/- standard error) per dysregulation-group bin (e.g., power_summary_fig5a.csv)
- Aggregated runtime means and standard deviations per method variant (e.g., table1_runtimes.csv)

## How to apply

Prepare gene expression data (e.g., breastcancer2012 or TCGA prostate) and estimate combined (weighted, directed) adjacency matrices per pathway with prepareAdjMat from the netgsa package, filtering input variables to keep only genes belonging to at least one tested pathway. Run NetGSA.REML.All on the full network to obtain test statistics and pathway-level calls; REML estimates variance components using residuals computed directly without evaluating the fixed effect coefficients. For benchmarking, time the combined runtime of network estimation and pathway enrichment with system-level timing and repeat runs to accumulate stable measurements. For statistical validation, simulate dysregulated-gene signals (mean signals of 0.2, 0.3, and 0.4 on dysregulated gene groups) and compare REML empirical power and type-I error against NetGSA.REHE with and without clustering, aggregating mean power within dysregulation-group bins (None, (0,5], (5,10], >10). REML is expected to be several times slower than REHE, with clustering (NetGSA.REHE.Cl) fastest, while the 'None' group mean power should be approximately 0 for all three methods.

## Related tools

- **netgsa** (R package providing NetGSA.REML.All, NetGSA.REHE, NetGSA.REHE.Cl, and prepareAdjMat for network estimation and pathway enrichment) — https://github.com/mikehellstern/netgsa
- **knitr** (Vignette/rmarkdown engine used to document and reproduce the netgsa workflow)

## Examples

```
library(netgsa); A <- prepareAdjMat(x, groups, pathways); fit <- NetGSA(A, x, y, pathways, method = "REML")
```

## Evaluation signals

- Mean power in the 'None' dysregulation group is approximately 0 for REML (and REHE variants), demonstrating type-I error control
- For dysregulated gene groups in bins (0,5], (5,10], >10, REML power should be comparable to (typically not exceeding) REHE with clustering, consistent with 'without sacrificing statistical power'
- Timing table shows NetGSA.REML.All runtime several times larger than NetGSA.REHE, with NetGSA.REHE.Cl fastest (Table 1 pattern)
- All input variables are filtered so only genes belonging to at least one tested pathway are passed to NetGSA

## Limitations

- REML on the entire network is computationally expensive; the paper's motivation is that NetGSA was previously 'difficult to use and computationally expensive'
- Adjacency matrices across groups may not share the same block diagonal structure, and the compatibility check between adjacency matrices and input data was removed in the current implementation ('this should be added later')
- The fixed effect coefficients beta are a byproduct; the README questions whether a better estimate of beta exists given the block diagonal structure of D

## Evidence

- [intro] we have completely overhauled the netgsa R package, an implementation of the NetGSA methodology: "To overcome these limitations, this paper provides important updates to an existing statistically powerful method, NetGSA"
- [readme] Variance components estimated directly from residuals: [evidence span withheld — anchor status: external_doc]
- [readme] Filter input data to pathway genes before NetGSA: [evidence span withheld — anchor status: external_doc]
- [abstract] Power without sacrificing statistical performance: "Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power"
- [other] Prepared via workflow comparison on breast cancer dataset: [evidence span withheld — anchor status: not_found]
