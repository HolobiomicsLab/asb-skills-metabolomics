---
name: network-data-export
description: Use when preparing gene-gene interaction networks for topology-based
  pathway enrichment (NetGSA) and input gene identifiers must be matched to pathway
  database edges without manual curation.
when_to_use_negative:
- The input adjacency matrices are already provided — NetGSA's main function accepts
  a user-supplied A (list of adjacency matrices across tested groups), so retrieval
  is unnecessary
- The goal is pathway enrichment scoring rather than network construction — use the
  NetGSA main function instead of edge-list retrieval
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0610
tools:
- name: netgsa
  role: Provides the obtainEdgeList() function that interfaces with graphite to retrieve
    pathway gene-gene interactions for topology-based pathway analysis
  repo: github.com/mikehellstern/netgsa
- name: KEGG
  role: User-specified external pathway database queried for pairwise gene-gene interactions
- name: Reactome
  role: User-specified external pathway database queried for pairwise gene-gene interactions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:631536c59a3f2f574784a48c7631a073624d526b5c6eef1d6ad40f9919b0280c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGG
  - Reactome
  - netgsa
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# network-data-export

## Summary

Retrieves gene-gene interaction (edge) information for user-specified pathway databases (e.g. KEGG, Reactome) via graphite and exports them as a standardized edge list, replacing the manual curation that previously made topology-based pathway analysis an expert task in netgsa.

## When to use

Use when preparing gene-gene interaction networks for topology-based pathway enrichment (NetGSA) and input gene identifiers must be matched to pathway database edges without manual curation.

## When NOT to use

- The input adjacency matrices are already provided — NetGSA's main function accepts a user-supplied A (list of adjacency matrices across tested groups), so retrieval is unnecessary
- The goal is pathway enrichment scoring rather than network construction — use the NetGSA main function instead of edge-list retrieval

## Inputs

- Gene identifier set (e.g. row names of the breastcancer2012 example dataset)
- User-specified pathway database names (e.g. c('kegg','reactome'))
- Identifier mapping resources (e.g. ENTREZID to UNIPROT via AnnotationDbi)

## Outputs

- Edge-list data frame honoring the 4-column contract (source node, target node, source identifier, target identifier), exportable as edge_list.csv

## How to apply

Start from the gene identifier set of the input data (e.g. the row names of the breastcancer2012 dataset shipped with netgsa) and specify the pathway databases of interest, such as c('kegg','reactome'). Query graphite to pull pairwise gene-gene interactions from each selected pathway database. Map pathway node identifiers into the same identifier space as the input genes (e.g. ENTREZID to UNIPROT via AnnotationDbi) and keep only edges whose both endpoints are present in the input gene set. Assemble the retained interactions into a 4-column edge-list contract: source node, target node, source identifier, target identifier. The rationale, per the paper, is that the overhauled netgsa 'removes the need for expert knowledge by directly curating gene-gene interaction information from multiple external databases', exposing this as the single R function obtainEdgeList().

## Related tools

- **netgsa** (Provides the obtainEdgeList() function that interfaces with graphite to retrieve pathway gene-gene interactions for topology-based pathway analysis) — github.com/mikehellstern/netgsa
- **KEGG** (User-specified external pathway database queried for pairwise gene-gene interactions)
- **Reactome** (User-specified external pathway database queried for pairwise gene-gene interactions)

## Examples

```
edgeList <- netgsa::obtainEdgeList(genes, c('kegg','reactome'))
```

## Evaluation signals

- Output file exists and parses as a CSV with exactly the 4 contracted columns (source node, target node, source identifier, target identifier)
- Every edge endpoint maps to an identifier present in the input gene set (e.g. breastcancer2012 row names) after identifier mapping
- Edges are restricted to interactions drawn from the user-specified pathway databases (kegg, reactome)
- Pathway node identifiers are successfully converted into the same identifier space as the input genes (e.g. ENTREZID to UNIPROT)

## Limitations

- Before running NetGSA, variables in the input data matrix should be filtered to keep only those belonging to at least one tested pathway (README note)
- The main NetGSA function no longer checks variable compatibility between adjacency matrices and input data for block-diagonal structures across groups — 'this should be added later' (README)
- Pathway interaction information is spread across several databases (KEGG, Reactome, PantherDB), so coverage depends on which databases the user specifies

## Evidence

- [abstract] removes the need for expert knowledge by directly curating gene-gene interaction information from multiple external databases: "removes the need for expert knowledge by directly curating gene-gene interaction information from multiple external databases"
- [intro] This information is often spread across several databases, such as KEGG, Reactome, and PantherDB: "This information is often spread across several databases, such as KEGG [10], Reactome [11], and PantherDB [12]"
- [readme] before running NetGSA, we should make sure to filter variables in the input data matrix to keep only those that belong to at least one tested pathway: [evidence span withheld — anchor status: external_doc]
