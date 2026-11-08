---
name: pathway-database-interaction-retrieval
description: Use when a topology-based pathway analysis needs gene-gene interaction
  (edge) lists pulled from pathway databases like KEGG or Reactome for a user-supplied
  gene identifier set, instead of hand-curated network files.
when_to_use_negative:
- The user already has a curated adjacency matrix or edge list in the same identifier
  space as their data — direct retrieval and mapping would be redundant.
- Genes of interest are not covered by any of the supported pathway databases (KEGG,
  Reactome, PantherDB), so graphite queries would return no usable interactions.
- The analysis does not require pathway structure (e.g. over-representation analysis
  with the entire pathway as a gene set), for which interaction retrieval is unnecessary.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: netgsa
  role: Provides obtainEdgeList(), the single R function exposing pathway database
    interaction retrieval for topology-based pathway enrichment.
  repo: github.com/mikehellstern/netgsa
- name: KEGG
  role: User-specified pathway database queried via graphite for pairwise gene-gene
    interactions.
- name: Reactome
  role: User-specified pathway database queried via graphite for pairwise gene-gene
    interactions.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:454f1b8d2ad2672adc99be5736fdece157ea08c8429ab6ba7651d554b8796f22
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGG
  - Reactome
  - netgsa
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pathway-database-interaction-retrieval

## Summary

Reconstructs the netgsa obtainEdgeList() retrieval step, which removes manual expert curation by querying graphite pathway databases (KEGG, Reactome, PantherDB) directly for gene-gene interaction information and returning a matched edge list in the user's identifier space.

## When to use

Use when a topology-based pathway analysis needs gene-gene interaction (edge) lists pulled from pathway databases like KEGG or Reactome for a user-supplied gene identifier set, instead of hand-curated network files.

## When NOT to use

- The user already has a curated adjacency matrix or edge list in the same identifier space as their data — direct retrieval and mapping would be redundant.
- Genes of interest are not covered by any of the supported pathway databases (KEGG, Reactome, PantherDB), so graphite queries would return no usable interactions.
- The analysis does not require pathway structure (e.g. over-representation analysis with the entire pathway as a gene set), for which interaction retrieval is unnecessary.

## Inputs

- Gene identifier vector (e.g. row names of the breastcancer2012 expression dataset)
- Vector of pathway database names, e.g. c('kegg','reactome')
- Identifier mapping resources (e.g. AnnotationDbi ENTREZID-to-UNIPROT maps)

## Outputs

- Edge-list data frame with a 4-column contract: source node, target node, source identifier, target identifier (e.g. saved as edge_list.csv)
- Edges restricted to endpoints present in the input gene set

## How to apply

Start from the row names of the input expression dataset (e.g. the breastcancer2012 example shipped with netgsa) as the gene identifier set. Call the obtainEdgeList mechanism against the user-specified pathway databases, e.g. c('kegg','reactome'), pulling pairwise gene-gene interactions from each pathway via graphite. Map pathway node identifiers (e.g. ENTREZID to UNIPROT using AnnotationDbi) so edges are expressed in the same identifier space as the input gene set, keeping only edges whose endpoints are present in the gene set — this mirrors the netgsa guidance to filter variables in the input matrix to those belonging to at least one tested pathway. Assemble matched interactions into a 4-column edge-list data frame (source node, target node, source identifier, target identifier). Rationale: the overhauled netgsa removes the need for expert knowledge by directly curating gene-gene interaction information from multiple external databases.

## Related tools

- **netgsa** (Provides obtainEdgeList(), the single R function exposing pathway database interaction retrieval for topology-based pathway enrichment.) — github.com/mikehellstern/netgsa
- **KEGG** (User-specified pathway database queried via graphite for pairwise gene-gene interactions.)
- **Reactome** (User-specified pathway database queried via graphite for pairwise gene-gene interactions.)

## Examples

```
edge_list <- obtainEdgeList(rownames(breastcancer2012), databases = c('kegg','reactome')); write.csv(edge_list, 'edge_list.csv', row.names = FALSE)
```

## Evaluation signals

- Output file exists and parses as a CSV with exactly the 4 contracted columns (source node, target node, source identifier, target identifier).
- Every edge endpoint corresponds to an identifier present in the input gene set (e.g. breastcancer2012 row names) after identifier mapping.
- Only user-specified databases (e.g. 'kegg' and 'reactome') contributed edges — no interactions from unrequested sources appear.
- Identifier mapping succeeded: no residual pathway-native identifiers (e.g. ENTREZID) remain where UNIPROT was requested.

## Limitations

- Variable filtering matters downstream: netgsa notes that before running NetGSA you should make sure to filter variables in the input data matrix to keep only those that belong to at least one tested pathway.
- Adjacency compatibility checks between pathway-derived networks and input data can be incomplete; the README notes a removed compatibility check with the caveat that 'this should be added later'.
- Retrieval depends on the pathway databases' coverage and identifier conventions (KEGG, Reactome, PantherDB), so interactions for genes outside these resources will be missing.

## Evidence

- [abstract] removes the need for expert knowledge by directly curating gene-gene interaction information from multiple external databases: "removes the need for expert knowledge by directly curating gene-gene interaction information from multiple external databases"
- [intro] This information is often spread across several databases, such as KEGG [10], Reactome [11], and PantherDB [12]: "This information is often spread across several databases, such as KEGG [10], Reactome [11], and PantherDB [12]"
- [readme] before running `NetGSA`, we should make sure to filter variables in the input data matrix to keep only those that belong to at least one tested pathway.: [evidence span withheld — anchor status: external_doc]
