---
name: r-bioconductor-programming
description: Use when a computational pipeline must be implemented, installed, or
  benchmarked in R — e.g. reproducing NetGSA REHE vs REML runtime comparisons or assembling
  weighted, directed pathway adjacency matrices with prepareAdjMat.
when_to_use_negative:
- The analysis requires methods like SPIA or PRS that need differentially expressed
  genes, or methods with specific input requirements such as topologyGSA or Pathway-Express
  — netgsa was overhauled to remove such constraints
- Only a single small pathway is of interest, where the REHE/clustering runtime optimizations
  provide negligible benefit over REML
- The task is interactive network visualization outside R, which is delegated to Cytoscape
  rather than scripted R code
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: netgsa
  role: 'R package being installed, exercised, and benchmarked: prepareAdjMat for
    adjacency estimation and NetGSA (REHE/REML variants) for enrichment'
  repo: github.com/mikehellstern/netgsa
- name: knitr
  role: Vignette engine used to build the package's executable documentation (knitr::rmarkdown
    vignettes)
- name: devtools
  role: GitHub installation of the netgsa package with vignettes built
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:674bdfe2fe43e9374b9245bce8c6ff5ecb87582d3b9d63e87be2c124a98fd88d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - devtools
  - knitr
  - netgsa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# r-bioconductor-programming

## Summary

Program in R to implement, benchmark, and document scientific workflows such as the overhauled netgsa package, whose REHE variance-component estimation and pathway clustering speed up topology-based pathway enrichment. The skill covers package installation from GitHub, vignette-driven documentation with knitr, and system-level runtime benchmarking.

## When to use

Use when a computational pipeline must be implemented, installed, or benchmarked in R — e.g. reproducing NetGSA REHE vs REML runtime comparisons or assembling weighted, directed pathway adjacency matrices with prepareAdjMat.

## When NOT to use

- The analysis requires methods like SPIA or PRS that need differentially expressed genes, or methods with specific input requirements such as topologyGSA or Pathway-Express — netgsa was overhauled to remove such constraints
- Only a single small pathway is of interest, where the REHE/clustering runtime optimizations provide negligible benefit over REML
- The task is interactive network visualization outside R, which is delegated to Cytoscape rather than scripted R code

## Inputs

- Gene expression datasets (e.g. breast cancer and TCGA prostate cancer, GEO/TCGA accessions)
- Curated pathway interaction data from databases such as KEGG, Reactome, PantherDB
- netgsa R package installed via devtools from GitHub

## Outputs

- Combined (weighted, directed) adjacency matrices per pathway from prepareAdjMat
- NetGSA enrichment results under REHE-with-clustering, REHE, and REML variants
- Aggregated runtime table (mean and SD per variant per dataset, e.g. table1_runtimes.csv)

## How to apply

Install the package directly from GitHub with devtools::install_github('mikehellstern/netgsa', build_vignettes=T) so the knitr-based vignettes ship with the code. Structure the analysis as an R workflow: load gene expression data (e.g. breast cancer and TCGA prostate cancer from GEO/TCGA), estimate weighted, directed adjacency matrices per pathway with prepareAdjMat, then run pathway enrichment variants (NetGSA.REHE.Cl, NetGSA.REHE, NetGSA.REML.All). Time each variant with R's system-level timing, repeating runs to accumulate stable measurements, and aggregate means and standard deviations into a table such as table1_runtimes.csv. Exploit the package's optimized internals — variance components estimated directly from residuals without evaluating fixed-effect coefficients, and contrast-vector products computed once via lapply — rather than reimplementing them.

## Related tools

- **netgsa** (R package being installed, exercised, and benchmarked: prepareAdjMat for adjacency estimation and NetGSA (REHE/REML variants) for enrichment) — github.com/mikehellstern/netgsa
- **knitr** (Vignette engine used to build the package's executable documentation (knitr::rmarkdown vignettes))
- **devtools** (GitHub installation of the netgsa package with vignettes built)

## Examples

```
library(devtools); devtools::install_github("mikehellstern/netgsa", build_vignettes=T)
```

## Evaluation signals

- The runtime table shows NetGSA.REHE several times faster than NetGSA.REML.All, with NetGSA.REHE.Cl fastest — an additional large runtime reduction from clustering
- Installation with build_vignettes=T completes and knitr vignettes are available locally
- prepareAdjMat returns a list of weighted, directed adjacency matrices matching the pathway blocks supplied
- Repeated timing runs yield aggregated means and standard deviations per variant per dataset that are stable across repeats
- Variables in the input data matrix are filtered to keep only those belonging to at least one tested pathway before calling NetGSA

## Limitations

- The compatibility check between adjacency matrices and input data across groups was removed; the user must verify block-diagonal structure themselves
- The README leaves open questions such as whether the beta fixed-effect output is needed and whether to assemble the entire adjacency matrix versus repeatedly subsetting variables
- Variance components require residuals, estimated directly without evaluating fixed-effect coefficients — input data must be filtered to pathway genes before running NetGSA

## Evidence

- [readme] devtools::install_github("mikehellstern/netgsa", build_vignettes=T): [evidence span withheld — anchor status: external_doc]
- [readme] Residuals are needed to estimate the variance components. This is done directly without evaluating the fixed effect coefficients.: [evidence span withheld — anchor status: external_doc]
- [intro] we have completely overhauled the netgsa R package, an implementation of the NetGSA methodology: "we have completely overhauled the netgsa R package, an implementation of the NetGSA methodology"
- [abstract] we have overhauled NetGSA, an existing topology-based method, to provide a computationally-efficient user-friendly tool: "we have overhauled NetGSA, an existing topology-based method, to provide a computationally-efficient user-friendly tool"
