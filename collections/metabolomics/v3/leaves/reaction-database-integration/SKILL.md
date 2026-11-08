---
name: reaction-database-integration
description: Use when building a knowledge-layer metabolic reaction network for annotation
  propagation and the available reaction databases are too sparse (e.g., only ~14,974
  reported reaction pairs) to connect metabolites.
when_to_use_negative:
- 'When you need a false-discovery-rate-controlled reaction set: no FDR is reported
  for the MRN because non-RPs are undefined under the screening conditions'
- When species-specific reaction coverage matters and BioTransformer's predefined
  reaction types would introduce biases for your organism
- When the task is annotating experimental features rather than building the knowledge
  layer (the data-driven MS2 similarity layer is built separately)
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: MetDNA3
  role: Consumes the curated MRN (knowledge layer) for network-based metabolite annotation;
    implements the two-layer networking strategy
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: igraph
  role: R library used to represent and assemble the reaction network as a graph object
- name: BioTransformer
  role: Generates unknown metabolites and their reaction links (990,441 generated
    RPs), constrained by predefined reaction types
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:a081899412b53def586f2f3e257e1499394bae75aa196fa815a745cf8524684b
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
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# reaction-database-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assemble a comprehensive metabolic reaction network (MRN) by integrating metabolite and reaction records from KEGG, MetaCyc, and HMDB into a seed network, then expanding it with GNN-predicted reaction pairs and BioTransformer-generated unknowns for use in network-based metabolite annotation.

## When to use

Use when building a knowledge-layer metabolic reaction network for annotation propagation and the available reaction databases are too sparse (e.g., only ~14,974 reported reaction pairs) to connect metabolites.

## When NOT to use

- When you need a false-discovery-rate-controlled reaction set: no FDR is reported for the MRN because non-RPs are undefined under the screening conditions
- When species-specific reaction coverage matters and BioTransformer's predefined reaction types would introduce biases for your organism
- When the task is annotating experimental features rather than building the knowledge layer (the data-driven MS2 similarity layer is built separately)

## Inputs

- Metabolite records from KEGG
- Metabolite and reaction records from MetaCyc
- Metabolite and reaction records from HMDB
- Candidate biotransformation Δmass matching results
- BioTransformer predictions of unknown metabolites and reaction links

## Outputs

- Curated metabolic reaction network (MRN) as an igraph object with 765,755 metabolites and 2,437,884 reaction pairs
- Seed knowledge network of 53,583 metabolites and 14,974 reported reaction pairs
- 1,432,469 GNN-predicted reaction pairs
- 990,441 BioTransformer-generated reaction pairs

## How to apply

First integrate metabolite and reaction records from KEGG, MetaCyc, and HMDB to assemble the seed knowledge network of 53,583 metabolites and 14,974 reported reaction pairs (RPs). Apply the two-step pre-screening strategy — Δmass matching of candidate biotransformations followed by a structural overlap threshold of ≥ 0.7 — to reduce false positives among candidate RPs. Train and apply a graph neural network (GNN) on the pre-screened candidate pairs to predict additional reaction relationships, adding 1,432,469 predicted RPs. Run BioTransformer to generate unknown metabolites and reaction links (990,441 generated RPs), noting its reliance on predefined reaction types. Assemble the full two-layer knowledge network as an igraph object; verify the final MRN contains exactly 765,755 metabolites and 2,437,884 reaction pairs (14,974 reported + 1,432,469 predicted + 990,441 generated).

## Related tools

- **MetDNA3** (Consumes the curated MRN (knowledge layer) for network-based metabolite annotation; implements the two-layer networking strategy) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **igraph** (R library used to represent and assemble the reaction network as a graph object)
- **BioTransformer** (Generates unknown metabolites and their reaction links (990,441 generated RPs), constrained by predefined reaction types)

## Examples

```
devtools::install_github("ZhuMetLab/MrnAnnoAlgo3")
```

## Evaluation signals

- Final MRN contains exactly 765,755 metabolites and 2,437,884 reaction pairs, with the reaction count decomposing as 14,974 reported + 1,432,469 GNN-predicted + 990,441 BioTransformer-generated
- Seed network contains exactly 53,583 metabolites and 14,974 reported reaction pairs before expansion
- Candidate reaction pairs pass both pre-screening steps: Δmass matching and structural overlap ≥ 0.7
- MRN topology shows substantially enhanced coverage and connectivity relative to the source knowledge databases
- The MRN assembles successfully into an igraph object consumable by MetDNA3's annotation propagation

## Limitations

- No FDR is reported for the MRN due to the lack of clearly defined non-RPs (negative examples) under the screening conditions and incompatibility of GNN performance metrics with the filtered candidate set
- GNN-based direct prediction of reaction pairs may include a considerable number of false positives
- BioTransformer-based generation of unknown metabolites has a limited success rate and may introduce species-specific biases due to its predefined reaction types

## Evidence
