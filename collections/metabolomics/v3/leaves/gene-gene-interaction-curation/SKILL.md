---
name: gene-gene-interaction-curation
description: 'Use when performing topology-based pathway enrichment and no pre-built
  pathway network is available: dataset gene lists (e.g. 2784 breast cancer genes)
  need gene-gene edges harvested from KEGG/Reactome and assembled into adjacency matrices.'
when_to_use_negative:
- You already have validated pathway adjacency matrices and only need to run the NetGSA
  statistical test
- You need differentially expressed gene lists rather than network-informed topology
  analysis — SPIA and PRS require detected DE genes and are computationally lighter
  alternatives
- Your genes belong to no tested pathway — the README advises filtering them out before
  assembling adjacency matrices
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: netgsa
  role: R package providing obtainEdgeList (edge curation from KEGG/Reactome) and
    prepareAdjMat with cluster=TRUE (adjacency matrix construction)
  repo: github.com/mikehellstern/netgsa
- name: NetGSA
  role: Main function consuming the curated adjacency matrices to fit the pathway
    enrichment model with lklMethod='REHE'
  repo: github.com/mikehellstern/netgsa
- name: KEGG
  role: External database from which gene-gene interaction edges are curated via obtainEdgeList
- name: Reactome
  role: External database from which gene-gene interaction edges are curated via obtainEdgeList
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:65eeed2f0e86eb80dfc042d5ab5499e8dea12ad7d42486e88841d3f5d544dd35
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# gene-gene-interaction-curation

## Summary

Curate gene-gene interaction edges from external pathway databases (KEGG, Reactome, PantherDB) and convert them into pathway adjacency matrices with within-pathway gene clustering, so topology-based NetGSA enrichment can run on thousands of genes without expert hand-built networks.

## When to use

Use when performing topology-based pathway enrichment and no pre-built pathway network is available: dataset gene lists (e.g. 2784 breast cancer genes) need gene-gene edges harvested from KEGG/Reactome and assembled into adjacency matrices.

## When NOT to use

- You already have validated pathway adjacency matrices and only need to run the NetGSA statistical test
- You need differentially expressed gene lists rather than network-informed topology analysis — SPIA and PRS require detected DE genes and are computationally lighter alternatives
- Your genes belong to no tested pathway — the README advises filtering them out before assembling adjacency matrices

## Inputs

- Matrix of gene expression data (e.g. breastcancer2012: 520 subjects x 2784 genes)
- List of gene identifiers for which edges must be curated
- External pathway databases: KEGG, Reactome (optionally PantherDB)
- Pathway membership definitions (e.g. 114 KEGG pathways)

## Outputs

- Edge list of curated gene-gene interactions per database
- List of pathway adjacency matrices across tested groups (block-diagonal, with within-pathway gene clustering)
- Pathway-level test statistics, p-values, and FDR q-values (pathway_tests[['results']] exportable as CSV)

## How to apply

Start from the measured gene list (e.g. the breastcancer2012 dataset with 2784 genes and 520 subjects) and call obtainEdgeList against KEGG and Reactome to curate gene-gene interaction edges, removing the need for expert knowledge of interaction databases. Build pathway adjacency matrices with prepareAdjMat using cluster=TRUE, which clusters genes within pathways to make the matrices computationally tractable for thousands of genes. Filter the input data matrix so it retains only genes belonging to at least one tested pathway, as the README notes. Feed the resulting list of (possibly block-diagonal) adjacency matrices into NetGSA with lklMethod='REHE' and optional sampling (sample_n=0.25, sample_p=0.25) to obtain test statistics, p-values, and FDR q-values per pathway.

## Related tools

- **netgsa** (R package providing obtainEdgeList (edge curation from KEGG/Reactome) and prepareAdjMat with cluster=TRUE (adjacency matrix construction)) — github.com/mikehellstern/netgsa
- **NetGSA** (Main function consuming the curated adjacency matrices to fit the pathway enrichment model with lklMethod='REHE') — github.com/mikehellstern/netgsa
- **KEGG** (External database from which gene-gene interaction edges are curated via obtainEdgeList)
- **Reactome** (External database from which gene-gene interaction edges are curated via obtainEdgeList)

## Examples

```
library(netgsa); edges <- obtainEdgeList(genes, databases=c('KEGG','Reactome')); A <- prepareAdjMat(x, genes, edges, cluster=TRUE); fit <- NetGSA(x, y, A, lklMethod='REHE', sampling=TRUE, sample_n=0.25, sample_p=0.25)
```

## Evaluation signals

- obtainEdgeList returns non-empty edge lists for all queried genes from both KEGG and Reactome
- prepareAdjMat(cluster=TRUE) produces adjacency matrices compatible (block-diagonal where applicable) with the input data columns
- Input data matrix filtered to retain only genes belonging to at least one tested pathway before NetGSA
- The downstream NetGSA REHE run with sampling (sample_n=0.25, sample_p=0.25) completes in minutes (reported ~2 minutes on a 2-CPU, 16 GB RAM machine)
- pathway_tests[['results']] table contains test statistics plus p-value and FDR q-value columns for all 114 tested KEGG pathways

## Limitations

- Curated edges are only as complete as KEGG/Reactome/PantherDB; networks may be incomplete for poorly characterized genes
- The README notes a removed compatibility check between adjacency matrices and input data when groups have differing block-diagonal structures, so users must validate matrix-data alignment manually
- prepareAdjMat with clustering changes within-pathway structure; eigenvalues via adj2inf do not change only for block-diagonal adjacency matrices
- Genes not belonging to any tested pathway must be filtered out of the input matrix before running NetGSA

## Evidence

- [abstract] removes the need for expert knowledge by directly curating gene-gene interaction information from multiple external databases: "removes the need for expert knowledge by directly curating gene-gene interaction information from multiple external databases"
- [methods] Curate gene-gene interaction edges with obtainEdgeList from the KEGG and Reactome databases for the genes in the dataset: [evidence span withheld — anchor status: not_found]
- [methods] Build pathway adjacency matrices with prepareAdjMat using cluster=TRUE to cluster genes within pathways.: [evidence span withheld — anchor status: not_found]
- [readme] before running `NetGSA`, we should make sure to filter variables in the input data matrix to keep only those that belong to at least one tested pathway.: [evidence span withheld — anchor status: external_doc]
- [intro] This information is often spread across several databases, such as KEGG [10], Reactome [11], and PantherDB [12]: "This information is often spread across several databases, such as KEGG [10], Reactome [11], and PantherDB [12]"
