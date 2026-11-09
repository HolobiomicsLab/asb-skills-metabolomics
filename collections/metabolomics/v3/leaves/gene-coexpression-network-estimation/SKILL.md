---
name: gene-coexpression-network-estimation
description: Use when you need topology-based pathway enrichment on gene expression
  data (e.g. GEO breast cancer or TCGA prostate cancer matrices) and must first estimate
  weighted, directed adjacency matrices per pathway from curated interaction databases.
when_to_use_negative:
- Your genes are not covered by any tested pathway — README advises filtering input
  data to genes belonging to at least one pathway before running NetGSA
- You need differential expression testing rather than topology-aware pathway enrichment;
  DE-only methods like SPIA or PRS may suffice and are computationally lighter
- You require a quick off-the-shelf enrichment without network estimation — over-representation
  methods do not need adjacency matrices
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0203
tools:
- name: netgsa
  role: R package providing prepareAdjMat for adjacency matrix estimation and NetGSA
    (with REHE/REML variants and clustering) for topology-based pathway enrichment
  repo: https://github.com/mikehellstern/netgsa
- name: knitr
  role: Vignette engine used to document the netgsa workflow (knitr::rmarkdown)
- name: Cytoscape
  role: Interactive visualization of the estimated pathway networks produced by netgsa
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:31c968e259f2407e865c1cc5e9d76162dff81e1b125459dc9466e0d1724ebfc1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikehellstern/netgsa
  tools:
  - Cytoscape
  - knitr
  - netgsa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# gene-coexpression-network-estimation

## Summary

Estimate combined (weighted, directed) gene interaction adjacency matrices for pathways from gene expression data using the netgsa package's prepareAdjMat, then use them for topology-based pathway enrichment with NetGSA. The overhauled netgsa makes this computationally feasible for thousands of genes via REHE variance component estimation and pathway clustering.

## When to use

Use when you need topology-based pathway enrichment on gene expression data (e.g. GEO breast cancer or TCGA prostate cancer matrices) and must first estimate weighted, directed adjacency matrices per pathway from curated interaction databases.

## When NOT to use

- Your genes are not covered by any tested pathway — README advises filtering input data to genes belonging to at least one pathway before running NetGSA
- You need differential expression testing rather than topology-aware pathway enrichment; DE-only methods like SPIA or PRS may suffice and are computationally lighter
- You require a quick off-the-shelf enrichment without network estimation — over-representation methods do not need adjacency matrices

## Inputs

- Gene expression matrix (e.g. GEO/TCGA breast cancer or prostate cancer expression data)
- Curated pathway/gene-gene interaction data (KEGG, Reactome, PantherDB)
- Pathway membership/annotation mapping genes to tested pathways

## Outputs

- Per-pathway combined weighted, directed adjacency matrices (list of adjacency matrices per group) from prepareAdjMat
- NetGSA pathway enrichment test statistics, degrees of freedom, and p-values
- Optional aggregated runtime comparison table (e.g. table1_runtimes.csv with mean and SD per variant per dataset)

## How to apply

Load the gene expression dataset (e.g. breast cancer or TCGA prostate cancer from GEO/TCGA) together with curated pathway interaction data (KEGG, Reactome, PantherDB) in R. Call netgsa's prepareAdjMat to estimate the combined weighted, directed adjacency matrix for each pathway. For enrichment, run NetGSA under the fast variants — NetGSA.REHE.Cl (REHE variance components with pathway clustering), NetGSA.REHE without clustering, or the slow NetGSA.REML.All — noting that REHE is several times faster than REML and clustering gives an additional large runtime reduction. Time variants with system-level timing and repeat to accumulate stable measurements if benchmarking. Before running NetGSA, filter the input data matrix to keep only genes belonging to at least one tested pathway, since the README explicitly recommends this pre-filtering step.

## Related tools

- **netgsa** (R package providing prepareAdjMat for adjacency matrix estimation and NetGSA (with REHE/REML variants and clustering) for topology-based pathway enrichment) — https://github.com/mikehellstern/netgsa
- **knitr** (Vignette engine used to document the netgsa workflow (knitr::rmarkdown))
- **Cytoscape** (Interactive visualization of the estimated pathway networks produced by netgsa)

## Examples

```
devtools::install_github("mikehellstern/netgsa", build_vignettes=T); A <- prepareAdjMat(x, group, pathways); fit <- NetGSA(x, group, pathways, A, variant="REHE.Cl")
```

## Evaluation signals

- Runtime check: NetGSA.REHE should be several times faster than NetGSA.REML.All, with NetGSA.REHE.Cl fastest, consistent with the paper's Table 1 timing results
- prepareAdjMat returns a list of adjacency matrices, one per pathway, with weighted directed edges
- Input data matrix is filtered to genes belonging to at least one tested pathway before NetGSA
- Enrichment runs on thousands of genes complete in minutes on a personal computer without sacrificing statistical power
- Aggregated runtimes (mean and SD per variant per dataset) are stable across repeated timed runs

## Limitations

- Compatibility checks between adjacency matrices and input data were removed for the block-diagonal structure and 'should be added later', so mismatched inputs may fail silently (README)
- Whether the fixed effect beta output from NetGSA is needed, or whether it should be estimated differently given the block-diagonal structure of D, remains open (README)
- Assembling the entire adjacency matrix vs. subsetting per pathway is an unresolved trade-off when working with many pathways (README)

## Evidence

- [abstract] Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power: "Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power"
- [full_text] workflow runtime comparison: [evidence span withheld — anchor status: not_found]
- [readme] README filtering advice: [evidence span withheld — anchor status: external_doc]
- [readme] README variance component optimization: [evidence span withheld — anchor status: external_doc]
