---
name: edge-list-construction
description: Use when building gene-gene interaction edge lists for topology-based
  pathway enrichment, starting from a gene identifier set (e.g. breastcancer2012 row
  names) and pathway databases such as 'kegg' and 'reactome'.
when_to_use_negative:
- The input data has not yet been filtered to keep only variables belonging to at
  least one tested pathway (the netgsa README notes variables in the input data matrix
  should be filtered to tested pathways before running NetGSA).
- The analysis goal is network visualization rather than interaction retrieval — Cytoscape-based
  visualization via netgsa is the appropriate path instead.
- The pathway interactions of interest are already available as a prepared adjacency
  matrix, in which case adjacency assembly (not database retrieval) is needed.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: netgsa
  role: R package whose obtainEdgeList() function exposes the pathway interaction
    retrieval capability through a single function
  repo: github.com/mikehellstern/netgsa
- name: KEGG
  role: External pathway database queried for gene-gene interaction information
- name: Reactome
  role: External pathway database queried for gene-gene interaction information
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:f13822bb39285255201f400d9a818d78ff9652b77d2ecd83c264fc07e9f2b1cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGG
  - Reactome
  - netgsa
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# edge-list-construction

## Summary

Reconstructs the netgsa obtainEdgeList retrieval step, which pulls pairwise gene-gene interactions from user-specified pathway databases (KEGG, Reactome) via graphite and assembles them into a 4-column edge list aligned to the input gene identifier space.

## When to use

Use when building gene-gene interaction edge lists for topology-based pathway enrichment, starting from a gene identifier set (e.g. breastcancer2012 row names) and pathway databases such as 'kegg' and 'reactome'.

## When NOT to use

- The input data has not yet been filtered to keep only variables belonging to at least one tested pathway (the netgsa README notes variables in the input data matrix should be filtered to tested pathways before running NetGSA).
- The analysis goal is network visualization rather than interaction retrieval — Cytoscape-based visualization via netgsa is the appropriate path instead.
- The pathway interactions of interest are already available as a prepared adjacency matrix, in which case adjacency assembly (not database retrieval) is needed.

## Inputs

- Gene identifier set (e.g. row names of the breastcancer2012 example dataset from the netgsa package)
- User-specified pathway databases, e.g. c('kegg','reactome')
- Identifier mapping resources (e.g. AnnotationDbi for ENTREZID to UNIPROT mapping)

## Outputs

- Edge-list data frame with exactly 4 columns: source node, target node, source identifier, target identifier
- edge_list.csv file containing the assembled edge list

## How to apply

Extract the gene identifier set from the input expression data (e.g. row names of the breastcancer2012 example dataset shipped with netgsa). Query the graphite pathway databases for the user-specified databases (c('kegg','reactome')), pulling pairwise gene-gene interactions from each pathway. Map pathway node identifiers to the input identifier space (e.g. ENTREZID to UNIPROT via AnnotationDbi) so edges match the breastcancer2012 row names, keeping only edges whose endpoints are present in the input gene set. Assemble the matched interactions into an edge-list data frame honoring obtainEdgeList's 4-column contract: source node, target node, source identifier, target identifier. This removes the manual curation that previously required expert effort, since netgsa now directly curates gene-gene interaction information from multiple external databases.

## Related tools

- **netgsa** (R package whose obtainEdgeList() function exposes the pathway interaction retrieval capability through a single function) — github.com/mikehellstern/netgsa
- **KEGG** (External pathway database queried for gene-gene interaction information)
- **Reactome** (External pathway database queried for gene-gene interaction information)

## Examples

```
library(netgsa); genes <- rownames(breastcancer2012); edge_list <- obtainEdgeList(genes, dbs=c('kegg','reactome')); write.csv(edge_list, 'edge_list.csv', row.names=FALSE)
```

## Evaluation signals

- The output file edge_list.csv exists and is parseable as a CSV.
- The CSV contains exactly the 4 contracted columns: source node, target node, source identifier, target identifier.
- Every edge endpoint corresponds to an identifier present in the breastcancer2012 row names after identifier mapping.
- Edges are drawn from the user-specified databases (kegg, reactome) and only pairwise interactions whose endpoints are in the vignette gene set are retained.

## Limitations

- The GitHub README (for mikehellstern/netgsa) is stale relative to the overhauled version — e.g. it still references the drjingma/netgsa Travis badge — so README details may lag the updated obtainEdgeList behavior.
- Identifier mismatch is a failure mode: pathway node identifiers (e.g. ENTREZID) must be mapped (e.g. to UNIPROT via AnnotationDbi) before matching against input gene identifiers, otherwise edges will be lost.
- Edges whose endpoints are not present in the input gene set are dropped, so the edge list is incomplete relative to the full pathway databases.

## Evidence

- [abstract] removes the need for expert knowledge by directly curating gene-gene interaction information from multiple external databases: "removes the need for expert knowledge by directly curating gene-gene interaction information from multiple external databases"
- [intro] This information is often spread across several databases, such as KEGG [10], Reactome [11], and PantherDB [12]: "This information is often spread across several databases, such as KEGG [10], Reactome [11], and PantherDB [12]"
- [readme] This package carries out Network-based Gene Set Analysis by incorporating external information about interactions among genes: "This package carries out Network-based Gene Set Analysis by incorporating external information about interactions among genes"
- [readme] before running `NetGSA`, we should make sure to filter variables in the input data matrix to keep only those that belong to at least one tested pathway: [evidence span withheld — anchor status: external_doc]
