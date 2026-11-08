---
name: gnn-reaction-pair-prediction
description: Use when building a knowledge-driven metabolic reaction network whose
  curated reaction pairs (e.g. ~14,974 from KEGG/MetaCyc/HMDB) are too sparse to support
  network-based metabolite annotation propagation.
when_to_use_negative:
- When the goal is annotation of experimental MS features directly — this skill only
  builds the knowledge layer; experimental data are pre-mapped via MS1 matching, reaction
  relationship mapping, and MS2 similarity constraints in a separate step
- When a formally validated FDR for the reaction network is required — the article
  explicitly refrains from reporting an FDR for the MRN because non-RPs are undefined
- When reaction space must be fully open-ended — BioTransformer-generated unknowns
  are limited by predefined reaction types and may introduce species-specific biases
edam_operation: http://edamontology.org/operation_2437
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: MetDNA3
  role: Implementation consuming the curated MRN as its knowledge layer for two-layer
    interactive annotation propagation
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: igraph
  role: R library used to assemble the full two-layer knowledge network as an igraph
    object
- name: BioTransformer
  role: Generates unknown metabolites and their reaction links (990,441 generated
    RPs) to complement GNN-predicted pairs
- name: KEGG
  role: Source database for seed metabolite and reaction records
- name: MetaCyc
  role: Source database for seed metabolite and reaction records
- name: HMDB
  role: Source database for seed metabolite and reaction records
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:772b44db38737d9f6766355b1e73beb164f9e89730fa5968d3bf33824c338a07
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  repo_url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  tools:
  - BioTransformer
  - HMDB
  - KEGG
  - MetDNA3
  - MetaCyc
  - igraph
  edam_operation: http://edamontology.org/operation_2437
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# GNN-based reaction pair prediction for metabolic reaction network curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains and applies a graph neural network to pre-screened candidate reaction pairs to predict additional metabolite-metabolite reaction relationships, expanding a KEGG/MetaCyc/HMDB-derived metabolic reaction network (MRN) beyond curated database knowledge for metabolite annotation in MetDNA3.

## When to use

Use when building a knowledge-driven metabolic reaction network whose curated reaction pairs (e.g. ~14,974 from KEGG/MetaCyc/HMDB) are too sparse to support network-based metabolite annotation propagation.

## When NOT to use

- When the goal is annotation of experimental MS features directly — this skill only builds the knowledge layer; experimental data are pre-mapped via MS1 matching, reaction relationship mapping, and MS2 similarity constraints in a separate step
- When a formally validated FDR for the reaction network is required — the article explicitly refrains from reporting an FDR for the MRN because non-RPs are undefined
- When reaction space must be fully open-ended — BioTransformer-generated unknowns are limited by predefined reaction types and may introduce species-specific biases

## Inputs

- Metabolite and reaction records from KEGG, MetaCyc, and HMDB (seed knowledge network)
- Candidate reaction pairs passing Δmass matching and structural overlap ≥ 0.7 pre-screening
- BioTransformer output (generated unknown metabolites and reaction links, optional)

## Outputs

- Curated metabolic reaction network (MRN) as an igraph object: 765,755 metabolites and 2,437,884 reaction pairs
- 1,432,469 GNN-predicted reaction pairs augmenting the 14,974 reported RPs
- Knowledge layer consumed by MetDNA3 annotation propagation

## How to apply

First assemble a seed knowledge network by integrating metabolite and reaction records from KEGG, MetaCyc, and HMDB (53,583 metabolites, 14,974 reported reaction pairs). Apply a two-step pre-screening strategy to candidate reaction pairs — Δmass matching of candidate biotransformations followed by a structural overlap threshold of ≥ 0.7 — to reduce potential false positives before training. Train the GNN on these pre-screened candidates and apply it to predict additional reaction relationships, adding 1,432,469 predicted RPs to the seed network. Optionally complement GNN predictions with BioTransformer-generated unknown metabolites and reaction links (990,441 generated RPs), noting its reliance on predefined reaction types. Assemble the resulting two-layer knowledge network as an igraph object so MetDNA3's annotation propagation can consume it; the final MRN should total 765,755 metabolites and 2,437,884 reaction pairs (14,974 reported + 1,432,469 predicted + 990,441 generated). Do not report an FDR for the MRN, since non-RPs are undefined under these screening conditions and GNN performance metrics are inapplicable to the filtered candidate set.

## Related tools

- **MetDNA3** (Implementation consuming the curated MRN as its knowledge layer for two-layer interactive annotation propagation) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **igraph** (R library used to assemble the full two-layer knowledge network as an igraph object)
- **BioTransformer** (Generates unknown metabolites and their reaction links (990,441 generated RPs) to complement GNN-predicted pairs)
- **KEGG** (Source database for seed metabolite and reaction records)
- **MetaCyc** (Source database for seed metabolite and reaction records)
- **HMDB** (Source database for seed metabolite and reaction records)

## Examples

```
if (!require("devtools")) install.packages("devtools"); devtools::install_github("ZhuMetLab/MrnAnnoAlgo3")
```

## Evaluation signals

- Final MRN must contain exactly 765,755 metabolites and 2,437,884 reaction pairs
- Reaction pair count must decompose exactly as 14,974 reported + 1,432,469 GNN-predicted + 990,441 BioTransformer-generated
- Seed knowledge network must match 53,583 metabolites and 14,974 reported reaction pairs before expansion
- Candidate reaction pairs must pass both Δmass matching and the structural overlap ≥ 0.7 threshold before GNN training
- Assembled network must be a valid igraph object consumable by MetDNA3's annotation propagation layer

## Limitations

- No FDR is reported for the curated MRN because non-RPs are undefined under the screening conditions and GNN performance metrics are inapplicable to the filtered candidate set
- GNN-based direct prediction of reaction pairs may include a considerable number of false positives
- BioTransformer-based generation of unknown metabolites has a limited success rate and may introduce species-specific biases due to predefined reaction types

## Evidence
