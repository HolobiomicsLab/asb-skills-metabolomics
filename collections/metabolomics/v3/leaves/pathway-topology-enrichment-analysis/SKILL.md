---
name: pathway-topology-enrichment-analysis
description: 'Use when you have a two-condition gene expression matrix (e.g., breastcancer2012:
  520 subjects × 2784 genes) and need topology-aware pathway enrichment against KEGG/Reactome
  interaction networks on ordinary hardware.'
when_to_use_negative:
- Your analysis only needs a list of differentially expressed genes fed into a fast
  method — SPIA and PRS are computationally efficient but require differentially expressed
  genes, whereas NetGSA does not.
- Your input genes are not covered by any interaction database in KEGG/Reactome/PantherDB,
  so obtainEdgeList cannot curate edges.
- You need a method with specific simple input requirements rather than network estimation
  (e.g., topologyGSA or Pathway-Express inputs).
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: netgsa
  role: 'R package implementing the overhauled NetGSA workflow: obtainEdgeList, prepareAdjMat
    (with clustering), and NetGSA (REHE + sampling)'
  repo: github.com/mikehellstern/netgsa
- name: NetGSA
  role: The topology-based enrichment method whose lklMethod='REHE' and sampling options
    enable fast, statistically powerful pathway testing
  repo: github.com/mikehellstern/netgsa
- name: KEGG
  role: External interaction database curated by obtainEdgeList to build gene-gene
    edges
- name: Reactome
  role: External interaction database curated by obtainEdgeList to build gene-gene
    edges
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:cbff0b916c87f7c5d0824f008ca86e644dbded11675f15c4099706fd754b87cf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGG
  - NetGSA
  - Pathway-Express
  - Reactome
  - knitr
  - netgsa
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# pathway-topology-enrichment-analysis

## Summary

Topology-based, third-generation pathway enrichment using the overhauled netgsa R package: curate gene-gene interactions from KEGG/Reactome, build clustered pathway adjacency matrices, and run NetGSA with REHE variance-component estimation and sampling so thousands of genes can be tested in minutes on a personal computer without sacrificing statistical power.

## When to use

Use when you have a two-condition gene expression matrix (e.g., breastcancer2012: 520 subjects × 2784 genes) and need topology-aware pathway enrichment against KEGG/Reactome interaction networks on ordinary hardware.

## When NOT to use

- Your analysis only needs a list of differentially expressed genes fed into a fast method — SPIA and PRS are computationally efficient but require differentially expressed genes, whereas NetGSA does not.
- Your input genes are not covered by any interaction database in KEGG/Reactome/PantherDB, so obtainEdgeList cannot curate edges.
- You need a method with specific simple input requirements rather than network estimation (e.g., topologyGSA or Pathway-Express inputs).

## Inputs

- Gene expression matrix with two biological conditions (e.g., breastcancer2012: 520 subjects, 2784 genes)
- Phenotype/group labels for subjects
- Pathway interaction databases: KEGG, Reactome (optionally PantherDB) for edge curation

## Outputs

- Pathway results table (pathway_tests[['results']]) with test statistics, p-values, and FDR q-values per pathway (114 KEGG pathways in the reference analysis)
- Clustered, weighted, directed adjacency matrices per pathway from prepareAdjMat
- CSV export of results (e.g., pathway_tests_results.csv) with recorded total runtime

## How to apply

Load the expression dataset (e.g., the packaged breastcancer2012 data with 520 subjects and 2784 genes) and curate gene-gene interaction edges with obtainEdgeList from KEGG and Reactome for the genes present in the data. Build pathway adjacency matrices with prepareAdjMat, setting cluster=TRUE so genes are clustered within pathways — this clustering is what yields the large additional runtime reduction on top of REHE. Fit the enrichment model with NetGSA using lklMethod='REHE' together with sampling=TRUE and sample_n=0.25, sample_p=0.25; REHE estimates variance components directly from residuals without evaluating fixed-effect coefficients, making it several times faster than REML on the full network. Extract pathway_tests[['results']] to obtain test statistics, p-values, and FDR q-values for all tested pathways (114 KEGG pathways in the reference run), and write the table (e.g., pathway_tests_results.csv) along with the total runtime.

## Related tools

- **netgsa** (R package implementing the overhauled NetGSA workflow: obtainEdgeList, prepareAdjMat (with clustering), and NetGSA (REHE + sampling)) — github.com/mikehellstern/netgsa
- **NetGSA** (The topology-based enrichment method whose lklMethod='REHE' and sampling options enable fast, statistically powerful pathway testing) — github.com/mikehellstern/netgsa
- **KEGG** (External interaction database curated by obtainEdgeList to build gene-gene edges)
- **Reactome** (External interaction database curated by obtainEdgeList to build gene-gene edges)

## Examples

```
devtools::install_github("mikehellstern/netgsa", build_vignettes=T); pt <- NetGSA(x, group, pathways, A, lklMethod="REHE", sampling=TRUE, sample_n=0.25, sample_p=0.25); results <- pt[["results"]]
```

## Evaluation signals

- The full pipeline (obtainEdgeList → prepareAdjMat(cluster=TRUE) → NetGSA with REHE and sampling) completes on the order of minutes — the reference run took ~2 minutes on a 2-CPU personal computer with 16 GB RAM.
- The results table contains p-value and FDR q-value columns for all tested pathways (114 KEGG pathways).
- A timing benchmark shows NetGSA.REHE several times faster than NetGSA.REML.All, with NetGSA.REHE.Cl (clustering) fastest in combined prepareAdjMat + NetGSA runtime.
- In the simulation power benchmark (FDR alpha=0.05), NetGSA.REHE.Cl is competitive with the top method across all dysregulated-gene-count groups.
- Before running NetGSA, variables in the input data matrix are filtered to keep only those belonging to at least one tested pathway.

## Limitations

- The README notes the compatibility check between adjacency matrices and input data was removed for block-diagonal adjacency structures and 'should be added later', so users must verify matrix-data consistency themselves.
- Input data must be pre-filtered: 'before running NetGSA, we should make sure to filter variables in the input data matrix to keep only those that belong to at least one tested pathway.'
- Adjacency matrices across groups are not assumed to share the same block diagonal structure, which affects how eigenvalues/adjacency lists (adj2inf) behave.

## Evidence

- [abstract] Existing tools limitation: "Existing software tools for topology-based pathway enrichment analysis are either computationally inefficient, have undesirable statistical power, or require expert knowledge"
- [abstract] Minutes on a personal computer: "Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power"
- [intro] Overhauled package: "we have completely overhauled the netgsa R package, an implementation of the NetGSA methodology"
- [intro] Interaction databases: "This information is often spread across several databases, such as KEGG [10], Reactome [11], and PantherDB [12]"
- [readme] Package purpose: "This package carries out Network-based Gene Set Analysis by incorporating external information about interactions among genes, as well as novel interactions learned from data."
