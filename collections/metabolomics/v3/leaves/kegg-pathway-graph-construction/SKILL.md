---
name: kegg-pathway-graph-construction
description: Use when preparing pathway-level kernel association tests (e.g., PaIRKAT)
  on metabolomics data where each metabolite must be assigned to a KEGG pathway graph
  with reaction-derived topology.
when_to_use_negative:
- The metabolite list lacks KEGG identifiers, so pathways cannot be populated with
  measured compounds
- The downstream analysis ignores pathway topography (unstructured kernel machine
  tests need no graph)
- Graph structure has not been examined and you intend to apply regularization blindly
  — the article notes it is important to examine graph structure before regularizing
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3172
tools:
- name: KEGGREST
  role: Fetches KEGG human pathway membership using the keggLink function to map metabolites
    to pathways
- name: igraph
  role: Builds each pathway's graph structure with edges defined by KEGG reaction
    database connections
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:effcb01047de356be360b069c75c715dddb46256bf56e7d43206ab8d7d08317b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CharlieCarpenter/PaIRKAT
  tools:
  - KEGGREST
  - R
  - igraph
  - impute (R package)
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# kegg-pathway-graph-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs KEGG pathway graphs for metabolomics kernel association testing: fetch human pathway membership with KEGGREST's keggLink, restrict to pathways with ≥10 measured metabolites, and build igraph graphs with edges defined by KEGG reaction database connections.

## When to use

Use when preparing pathway-level kernel association tests (e.g., PaIRKAT) on metabolomics data where each metabolite must be assigned to a KEGG pathway graph with reaction-derived topology.

## When NOT to use

- The metabolite list lacks KEGG identifiers, so pathways cannot be populated with measured compounds
- The downstream analysis ignores pathway topography (unstructured kernel machine tests need no graph)
- Graph structure has not been examined and you intend to apply regularization blindly — the article notes it is important to examine graph structure before regularizing

## Inputs

- Metabolomics feature table with KEGG compound identifiers (e.g., COPDGene PR000907 metabolite list)
- KEGG human pathway membership links via KEGGREST keggLink
- KEGG reaction database connections

## Outputs

- Per-pathway igraph graph objects (one per qualifying pathway, e.g., 28 KEGG pathways)
- Pathway-to-metabolite membership mapping restricted to pathways with ≥10 measured metabolites

## How to apply

Fetch KEGG human pathway membership using the keggLink function from the KEGGREST package, matching metabolites that have KEGG identifiers (436 of the COPDGene metabolites had KEGG ids). Restrict to pathways with measurements on at least 10 metabolites — in the COPDGene application this yielded 28 KEGG pathways. For each retained pathway, build an igraph graph whose edges are defined by connections within the pathway from the KEGG reaction database; these graphs become the kernel topography for downstream normalized Laplacian or PaIRKAT regularized score tests. Inspect graph structure before applying regularization, since sparse connectivity and disconnected pathways hinder test performance.

## Related tools

- **KEGGREST** (Fetches KEGG human pathway membership using the keggLink function to map metabolites to pathways)
- **igraph** (Builds each pathway's graph structure with edges defined by KEGG reaction database connections)

## Examples

```
library(KEGGREST); library(igraph); pw <- keggLink("pathway", "hsa"); g <- graph_from_data_frame(reaction_edges, directed = FALSE)
```

## Evaluation signals

- The retained pathway set matches the expected count (28 KEGG pathways with measurements on at least 10 metabolites in the COPDGene application)
- Every pathway graph has ≥10 metabolite nodes drawn from the filtered dataset
- Each igraph graph's edges trace to KEGG reaction database connections within the pathway, not arbitrary ties
- Graph connectivity is inspected before regularization; disconnected graphs are flagged since the unregularized normalized Laplacian hinders performance on them
- The full pipeline (graphs + PaIRKAT with tau = 1) reproduces the reported power pattern: improved testing power for 18/28 pathways for FEV1/FVC and 17/28 for percent emphysema

## Limitations

- Sparse connectivity in many metabolomic pathways can greatly hinder performance of graph-kernel tests
- The filtered dataset may not contain every metabolite in a selected KEGG pathway, so some analyzed pathways can have fewer than their nominal metabolite count
- Graph structure must be examined before applying regularization; the method is not universally applicable without inspection
- Disjoint (disconnected) pathways motivated the regularization step — the unregularized normalized Laplacian hinders testing performance on disconnected graphs

## Evidence

- [discussion] Edges defined by KEGG reaction database connections: [evidence span withheld — anchor status: not_found]
