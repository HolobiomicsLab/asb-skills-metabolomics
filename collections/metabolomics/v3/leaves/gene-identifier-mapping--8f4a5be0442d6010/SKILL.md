---
name: gene-identifier-mapping
description: Use when pathway node identifiers retrieved from databases like KEGG
  or Reactome must be matched to the identifiers of your input gene set (e.g. ENTREZID
  to UNIPROT) before assembling edge lists for topology-based pathway analysis.
when_to_use_negative:
- Input gene identifiers already match the pathway node identifier space, so no mapping
  step is needed
- The analysis goal is pathway enrichment on an already-built adjacency matrix rather
  than retrieving/matching edges from external pathway databases
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: netgsa
  role: R package whose obtainEdgeList function exposes the edge-list retrieval and
    identifier-matching workflow
  repo: github.com/mikehellstern/netgsa
- name: graphite
  role: Interface queried for gene-gene interaction information from user-specified
    pathway databases (KEGG, Reactome)
- name: AnnotationDbi
  role: Used to map pathway node identifiers (e.g. ENTREZID) to the input identifier
    space (e.g. UNIPROT)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:d13bc61c93becd305d901914c3fe1db4eadd249520d08b9cc1f9fc6f36635fec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnotationDbi
  - KEGG
  - Reactome
  - graphite
  - netgsa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gene-identifier-mapping

## Summary

Maps pathway node identifiers from external interaction databases (KEGG, Reactome) into the same identifier space as the input gene set (e.g. ENTREZID to UNIPROT via AnnotationDbi) so retrieved gene-gene edges align with the user's data, as done by netgsa's obtainEdgeList workflow.

## When to use

Use when pathway node identifiers retrieved from databases like KEGG or Reactome must be matched to the identifiers of your input gene set (e.g. ENTREZID to UNIPROT) before assembling edge lists for topology-based pathway analysis.

## When NOT to use

- Input gene identifiers already match the pathway node identifier space, so no mapping step is needed
- The analysis goal is pathway enrichment on an already-built adjacency matrix rather than retrieving/matching edges from external pathway databases

## Inputs

- Gene identifier vector from the input data matrix (e.g. row names of the breastcancer2012 dataset)
- Pathway database specification (e.g. c('kegg','reactome'))
- Identifier-mapping resources such as ENTREZID-to-UNIPROT maps via AnnotationDbi

## Outputs

- Edge-list data frame with 4 columns (source node, target node, source identifier, target identifier), e.g. saved as edge_list.csv
- Edges restricted to identifiers present in the input gene set

## How to apply

Start from the row names of the input data matrix (e.g. breastcancer2012) to define the target gene identifier set. Query graphite for pairwise gene-gene interactions from user-specified pathway databases such as c('kegg','reactome'). Map pathway node identifiers to the input identifier space (e.g. ENTREZID to UNIPROT via AnnotationDbi), then keep only edges whose endpoints are present in the input gene set. Assemble matched interactions into an edge-list honoring obtainEdgeList's 4-column contract (source node, target node, source identifier, target identifier). Rationale: the overhauled netgsa removes the need for expert knowledge by directly curating gene-gene interaction information from multiple external databases, but edges must be expressed in the input identifier space for downstream topology-based pathway enrichment.

## Related tools

- **netgsa** (R package whose obtainEdgeList function exposes the edge-list retrieval and identifier-matching workflow) — github.com/mikehellstern/netgsa
- **graphite** (Interface queried for gene-gene interaction information from user-specified pathway databases (KEGG, Reactome))
- **AnnotationDbi** (Used to map pathway node identifiers (e.g. ENTREZID) to the input identifier space (e.g. UNIPROT))

## Examples

```
edges <- netgsa::obtainEdgeList(rownames(breastcancer2012), databases=c('kegg','reactome'))
```

## Evaluation signals

- Output file edge_list.csv exists and parses as a CSV with exactly the 4 contracted columns (source node, target node, source identifier, target identifier)
- Every edge endpoint corresponds to an identifier present in the breastcancer2012 row names after mapping
- No unmapped pathway nodes remain in the edge list (all identifiers share one namespace)
- Edges are restricted to the user-specified databases (c('kegg','reactome')) and the vignette gene set

## Limitations

- Edges are discarded when either endpoint is absent from the input gene set after mapping, so partially matched interactions are lost
- Identifier mapping depends on AnnotationDbi cross-references; missing or ambiguous ENTREZID-to-UNIPROT mappings can silently drop edges
- The README warns that before running NetGSA, variables in the input data matrix should be filtered to keep only those belonging to at least one tested pathway

## Evidence

- [abstract] abstract: "removes the need for expert knowledge by directly curating gene-gene interaction information from multiple external databases"
- [other] other: [evidence span withheld — anchor status: not_found]
- [readme] readme: "This package carries out Network-based Gene Set Analysis by incorporating external information about interactions among genes"
