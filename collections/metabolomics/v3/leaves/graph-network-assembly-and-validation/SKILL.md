---
name: graph-network-assembly-and-validation
description: Use when building or sanity-checking a knowledge-driven reaction network
  from metabolite databases plus computational reaction prediction, e.g. to expand
  coverage beyond reported reaction pairs (RPs).
when_to_use_negative:
- The goal is a data-driven MS2 spectral-similarity network (e.g. GNPS/FBMN) rather
  than a knowledge-driven reaction network
- A defensible FDR estimate for reaction pairs is required — the article explicitly
  refrains from reporting one because non-RPs are undefined under the screening conditions
- The task needs species-specific or novel reaction types beyond BioTransformer's
  predefined reaction set
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetDNA3 (MrnAnnoAlgo3)
  role: Core algorithm module that consumes the assembled igraph MRN knowledge layer
    for two-layer annotation propagation
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: igraph
  role: R graph library used to represent the assembled two-layer knowledge network
- name: BioTransformer
  role: Generates unknown metabolites and their reaction links to expand the MRN beyond
    reported and GNN-predicted pairs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:bf5dc55a303c6209740991ba52a5337d22a929013fd1ebc04df5473e086ab292
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
  - MetDNA3 (MrnAnnoAlgo3)
  - MetaCyc
  - igraph
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# graph-network-assembly-and-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles and validates a metabolic reaction network (MRN) by integrating KEGG, MetaCyc, and HMDB seed records with GNN-predicted and BioTransformer-generated reaction pairs, and represents the result as an igraph object for downstream annotation propagation.

## When to use

Use when building or sanity-checking a knowledge-driven reaction network from metabolite databases plus computational reaction prediction, e.g. to expand coverage beyond reported reaction pairs (RPs).

## When NOT to use

- The goal is a data-driven MS2 spectral-similarity network (e.g. GNPS/FBMN) rather than a knowledge-driven reaction network
- A defensible FDR estimate for reaction pairs is required — the article explicitly refrains from reporting one because non-RPs are undefined under the screening conditions
- The task needs species-specific or novel reaction types beyond BioTransformer's predefined reaction set

## Inputs

- Metabolite and reaction records from KEGG, MetaCyc, and HMDB
- Candidate biotransformation pairs with molecular Δmass values
- Structural fingerprints for computing structural overlap of candidate RPs
- Pre-screened candidate reaction pairs for GNN training/inference
- Species context for BioTransformer biotransformation generation

## Outputs

- Curated MRN as an igraph object with 765,755 metabolites and 2,437,884 reaction pairs
- Layered RP provenance: 14,974 reported, 1,432,469 GNN-predicted, 990,441 BioTransformer-generated
- Knowledge-layer network consumed by MetDNA3 annotation propagation

## How to apply

First, integrate metabolite and reaction records from KEGG, MetaCyc, and HMDB to build the seed knowledge network (in this study 53,583 metabolites and 14,974 reported RPs). Second, apply a two-step pre-screening strategy to candidate biotransformations — Δmass matching followed by a structural overlap threshold of ≥ 0.7 — to reduce false positives. Third, train and apply a graph neural network (GNN) on the pre-screened candidates to add predicted RPs (1,432,469 added here), and run BioTransformer to generate unknown metabolites and their reaction links (990,441 generated RPs here). Finally, assemble the full two-layer knowledge network as an igraph object for MetDNA3's annotation propagation, and validate counts as a sum invariant: 14,974 reported + 1,432,469 predicted + 990,441 generated = 2,437,884 RPs over 765,755 metabolites.

## Related tools

- **MetDNA3 (MrnAnnoAlgo3)** (Core algorithm module that consumes the assembled igraph MRN knowledge layer for two-layer annotation propagation) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **igraph** (R graph library used to represent the assembled two-layer knowledge network)
- **BioTransformer** (Generates unknown metabolites and their reaction links to expand the MRN beyond reported and GNN-predicted pairs)

## Examples

```
if (!require("devtools")) install.packages("devtools"); devtools::install_github("ZhuMetLab/MrnAnnoAlgo3")
```

## Evaluation signals

- Count invariant: total metabolites = 765,755 and total RPs = 2,437,884, with per-layer sums 14,974 + 1,432,469 + 990,441 matching the total exactly
- Seed network matches source databases: 53,583 metabolites and 14,974 reported RPs from KEGG/MetaCyc/HMDB
- Every GNN-predicted candidate passed the two-step pre-screen (Δmass matching plus structural overlap ≥ 0.7) before inclusion
- Network is loadable as an igraph object and usable as MetDNA3's knowledge layer for annotation propagation
- Coverage and topological connectivity are substantially higher than the source knowledge databases

## Limitations

- No FDR is reported for the curated MRN because non-RPs (negative examples) are undefined under the screening conditions, and GNN performance metrics do not transfer to the filtered candidate set
- Direct GNN prediction of reaction pairs may include a considerable number of false positives
- BioTransformer generation of unknown metabolites has a limited success rate and may introduce species-specific biases due to its predefined reaction types

## Evidence

- [discussion] due to the lack of clearly defined non-RPs under these screening conditions, and the inapplicability of applying GNN-based performance metrics to the filtered candidate set: [evidence span withheld — anchor status: not_found]
