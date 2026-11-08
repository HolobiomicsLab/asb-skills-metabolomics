---
name: restricted-likelihood-pathway-testing
description: Use when performing topology-based pathway enrichment on gene-level expression
  data (e.g., 2784 genes, 520 subjects) where standard NetGSA is computationally expensive
  and you need efficient REHE-based testing.
when_to_use_negative:
- Your method requires differentially expressed genes as input and you prefer lightweight
  tools like SPIA or PRS instead of a topology-based approach
- Input variables have not been filtered to those belonging to at least one tested
  pathway — the README advises filtering the data matrix to pathway members before
  running NetGSA
- You need interactive network visualization as the primary goal rather than statistical
  testing (use the Cytoscape integration instead)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: netgsa
  role: R package providing obtainEdgeList, prepareAdjMat (with clustering), and the
    NetGSA function run with lklMethod='REHE' and sampling
  repo: github.com/mikehellstern/netgsa
- name: NetGSA
  role: Overhauled topology-based pathway enrichment method whose REHE likelihood
    and sampling options make testing computationally efficient
  repo: github.com/mikehellstern/netgsa
- name: KEGG
  role: External database curated by obtainEdgeList for gene-gene interaction edges
- name: Reactome
  role: External database curated by obtainEdgeList for gene-gene interaction edges
- name: knitr
  role: Vignette engine used to document the netgsa workflow (knitr::rmarkdown)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:40ed3f422ec979dea75cd9b1c805804a7bd920f968092c15ffbba26c71c9b5aa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGG
  - NetGSA
  - Reactome
  - knitr
  - netgsa
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# restricted-likelihood-pathway-testing

## Summary

Fits topology-based pathway enrichment with the overhauled NetGSA using restricted likelihood (REHE) variance-component estimation plus sampling so tests over thousands of genes finish in minutes on a personal computer without sacrificing statistical power.

## When to use

Use when performing topology-based pathway enrichment on gene-level expression data (e.g., 2784 genes, 520 subjects) where standard NetGSA is computationally expensive and you need efficient REHE-based testing.

## When NOT to use

- Your method requires differentially expressed genes as input and you prefer lightweight tools like SPIA or PRS instead of a topology-based approach
- Input variables have not been filtered to those belonging to at least one tested pathway — the README advises filtering the data matrix to pathway members before running NetGSA
- You need interactive network visualization as the primary goal rather than statistical testing (use the Cytoscape integration instead)

## Inputs

- Gene-level expression dataset such as the packaged breastcancer2012 dataset (520 subjects, 2784 genes)
- External pathway/interaction databases: KEGG and Reactome (also PantherDB mentioned as a source)

## Outputs

- pathway_tests[['results']] table with test statistics, p-values, and FDR q-values per pathway (e.g., 114 KEGG pathways)
- Pathway adjacency matrices (block-diagonal, clustered within pathways) from prepareAdjMat
- Total pipeline runtime (expected on the order of minutes)

## How to apply

Curate gene-gene interaction edges with obtainEdgeList from KEGG and Reactome for the genes in the dataset, then build pathway adjacency matrices with prepareAdjMat using cluster=TRUE to cluster genes within pathways. Fit the enrichment model by calling NetGSA with lklMethod='REHE' and sampling=TRUE with sample_n=0.25 and sample_p=0.25, which estimates variance components directly from residuals without evaluating fixed-effect coefficients and reuses contrast-vector products for degrees of freedom and test statistics. Extract pathway_tests[['results']] containing test statistics, p-values, and FDR q-values for all tested pathways (114 KEGG pathways in the breast cancer 2012 example). Rationale: REHE with sampling lets pathway enrichment for thousands of genes run in minutes on ordinary hardware (about 2 minutes on a 2-CPU, 16 GB RAM machine) while retaining statistical power.

## Related tools

- **netgsa** (R package providing obtainEdgeList, prepareAdjMat (with clustering), and the NetGSA function run with lklMethod='REHE' and sampling) — github.com/mikehellstern/netgsa
- **NetGSA** (Overhauled topology-based pathway enrichment method whose REHE likelihood and sampling options make testing computationally efficient) — github.com/mikehellstern/netgsa
- **KEGG** (External database curated by obtainEdgeList for gene-gene interaction edges)
- **Reactome** (External database curated by obtainEdgeList for gene-gene interaction edges)
- **knitr** (Vignette engine used to document the netgsa workflow (knitr::rmarkdown))

## Examples

```
pathway_tests <- NetGSA(x, group, pathways, A, lklMethod = 'REHE', sampling = TRUE, sample_n = 0.25, sample_p = 0.25)
```

## Evaluation signals

- Pipeline completes on the order of minutes — consistent with the reported ~2 minutes on a 2-CPU, 16 GB personal computer
- Results table contains p-value and FDR q-value columns for all tested pathways (e.g., 114 KEGG pathways)
- Adjacency matrices from prepareAdjMat are block-diagonal per group, and data are filtered to genes in at least one tested pathway before NetGSA
- Sampling parameters sample_n=0.25 and sample_p=0.25 are applied with lklMethod='REHE' without loss of statistical power relative to full computation
- Runtime and output are reproducible when written to pathway_tests_results.csv

## Limitations

- The compatibility check between adjacency matrices and input data was removed for non-shared block-diagonal structure across groups and 'should be added later'
- Whether the fixed-effect coefficients beta output from NetGSA is needed, or could be better estimated given the block-diagonal structure of D, remains an open question in the README
- Assembling the entire adjacency matrix versus subsetting per pathway is unresolved when many pathways are tested, to avoid repeatedly subsetting variables
- Variables in the input data matrix must be filtered to those belonging to at least one tested pathway before running NetGSA

## Evidence

- [abstract] Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power: "Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power"
- [results] Running the netgsa workflow on the breast cancer 2012 dataset with REHE and sampling (sample_n = 0.25, sample_p = 0.25) took about 2 minutes on a 2 CPU personal computer with 16 GB of RAM: [evidence span withheld — anchor status: not_found]
- [methods] Fit the pathway enrichment model with NetGSA using lklMethod='REHE' and sampling=TRUE with sample_n=0.25 and sample_p=0.25: [evidence span withheld — anchor status: not_found]
- [readme] before running `NetGSA`, we should make sure to filter variables in the input data matrix to keep only those that belong to at least one tested pathway: [evidence span withheld — anchor status: external_doc]
- [readme] Variance component estimation: Residuals are needed to estimate the variance components. This is done directly without evaluating the fixed effect coefficients.: [evidence span withheld — anchor status: external_doc]
