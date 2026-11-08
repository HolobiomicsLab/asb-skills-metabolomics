---
name: in-silico-metabolite-biotransformation-generation
description: Use when a knowledge-driven metabolic reaction network built from KEGG,
  MetaCyc, and HMDB is too sparse, and putative metabolites/reaction pairs absent
  from those databases are needed to improve annotation coverage and topological connectivity.
when_to_use_negative:
- When the goal is FDR-controlled validation of already-annotated features — no FDR
  is reported for the MRN because non-RPs are undefined under the screening conditions
- When data- and knowledge-driven coverage is already sufficient and additional generated
  reaction pairs would only add unquantified false positives
- When species-specific biotransformations outside BioTransformer's predefined reaction
  types are required
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: BioTransformer
  role: Generates unknown metabolites and their reaction links (990,441 generated
    RPs) to expand the seed knowledge network; relies on predefined reaction types
- name: MetDNA3
  role: Consumes the igraph knowledge-layer network containing the BioTransformer-generated
    pairs for recursive metabolite annotation propagation
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: igraph
  role: Assembles the full two-layer knowledge network object (metabolites and reaction
    pairs) consumed by MetDNA3
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:95c24fd74b58f4b49e1ee49eda5b217597f4ac825820368f4ee739892d0616ae
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# in-silico-metabolite-biotransformation-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Uses BioTransformer to generate unknown metabolites and their reaction links beyond KEGG/MetaCyc/HMDB coverage, contributing 990,441 generated reaction pairs to MetDNA3's curated metabolic reaction network (765,755 metabolites, 2,437,884 reaction pairs).

## When to use

Use when a knowledge-driven metabolic reaction network built from KEGG, MetaCyc, and HMDB is too sparse, and putative metabolites/reaction pairs absent from those databases are needed to improve annotation coverage and topological connectivity.

## When NOT to use

- When the goal is FDR-controlled validation of already-annotated features — no FDR is reported for the MRN because non-RPs are undefined under the screening conditions
- When data- and knowledge-driven coverage is already sufficient and additional generated reaction pairs would only add unquantified false positives
- When species-specific biotransformations outside BioTransformer's predefined reaction types are required

## Inputs

- Seed metabolite structures and reaction records integrated from KEGG, MetaCyc, and HMDB (53,583 metabolites, 14,974 reported reaction pairs)
- Pre-screened candidate reaction pairs passing Δmass matching and structural overlap ≥ 0.7

## Outputs

- BioTransformer-generated unknown metabolites
- 990,441 BioTransformer-generated reaction pairs
- Full curated MRN as an igraph object (765,755 metabolites, 2,437,884 reaction pairs) serving as the knowledge layer for MetDNA3

## How to apply

Start from the seed knowledge network assembled by integrating metabolite and reaction records from KEGG, MetaCyc, and HMDB (53,583 metabolites, 14,974 reported reaction pairs). Run BioTransformer on these seed metabolites to enumerate unknown metabolites and their biotransformation links, which contributes 990,441 generated reaction pairs. Apply the same two-step pre-screening used for other candidate reaction pairs — Δmass matching of candidate biotransformations followed by a structural overlap threshold of ≥ 0.7 — to reduce potential false positives. Merge the generated pairs with 14,974 reported pairs and 1,432,469 GNN-predicted pairs, then assemble the full two-layer knowledge network as an igraph object that MetDNA3 consumes for annotation propagation. Do not attempt to report an FDR for this expansion: non-reaction pairs (negative examples) are undefined under the screening conditions, and BioTransformer's predefined reaction types may introduce species-specific biases.

## Related tools

- **BioTransformer** (Generates unknown metabolites and their reaction links (990,441 generated RPs) to expand the seed knowledge network; relies on predefined reaction types)
- **MetDNA3** (Consumes the igraph knowledge-layer network containing the BioTransformer-generated pairs for recursive metabolite annotation propagation) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **igraph** (Assembles the full two-layer knowledge network object (metabolites and reaction pairs) consumed by MetDNA3)

## Examples

```
devtools::install_github("ZhuMetLab/MrnAnnoAlgo3")
```

## Evaluation signals

- Final MRN contains exactly 765,755 metabolites and 2,437,884 reaction pairs (14,974 reported + 1,432,469 predicted + 990,441 BioTransformer-generated)
- BioTransformer-generated RPs contribute exactly 990,441 pairs to the total
- Every generated candidate pair passed the two-step pre-screening: Δmass matching plus structural overlap ≥ 0.7
- Knowledge network is represented as an igraph object usable as MetDNA3's knowledge layer for annotation propagation
- No FDR is claimed for the generated pairs, consistent with non-RPs being undefined under the screening conditions

## Limitations

- BioTransformer-based generation of unknown metabolites has a limited success rate
- Generation is limited by predefined reaction types, potentially introducing species-specific biases
- No FDR can be reported for the MRN due to lack of clearly defined non-RPs under the screening conditions
- GNN-based direct prediction of reaction pairs may include a considerable number of false positives (affects the merged network)

## Evidence

- [methods] Run BioTransformer to generate unknown metabolites and their reaction links, contributing 990,441 generated RPs: [evidence span withheld — anchor status: not_found]
