---
name: metabolic-reaction-network-curation
description: Use when building or expanding a knowledge-driven metabolic reaction
  network from KEGG/MetaCyc/HMDB records with predicted reaction pairs, or when verifying
  curated MRN metabolite/reaction-pair counts.
when_to_use_negative:
- Input is already an annotated feature table and the goal is annotation propagation
  rather than network construction (use the MRN as a knowledge layer instead of re-curation)
- You require a validated FDR for the MRN itself — no FDR is reported because non-RPs
  are undefined under the screening conditions
- The analysis is purely statistical/clustering over features with no biochemical
  reaction knowledge involved
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetDNA3 (MrnAnnoAlgo3)
  role: Core algorithm module providing MRN construction, GNN-based reaction prediction,
    and igraph-based knowledge-layer assembly
  repo: github.com/ZhuMetLab/MrnAnnoAlgo3
- name: BioTransformer
  role: Generates unknown metabolites and their reaction links (990,441 generated
    reaction pairs), limited by predefined reaction types
- name: igraph
  role: Assembles and stores the curated two-layer MRN as a graph object
- name: SpectraTools
  role: Computes modified dot-product MS2 spectral similarity (via getMatchParam)
    to filter reaction-linked feature pairs at ≥ 0.5 into the data-constrained MRN
  repo: https://github.com/ZhuMetLab/SpectraTools
- name: KEGG / MetaCyc / HMDB
  role: Seed knowledge sources supplying the 53,583 metabolites and 14,974 reported
    reaction pairs
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:b1175c54bb20b9c70a812a7d7b87777664654b20ebe4f918e455e42c18646f8e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  tools:
  - BioTransformer
  - HMDB
  - KEGG
  - KEGG / MetaCyc / HMDB
  - MetDNA3
  - MetDNA3 (MrnAnnoAlgo3)
  - MetaCyc
  - SpectraTools
  - igraph
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabolic-reaction-network-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curates a comprehensive metabolic reaction network (MRN) by integrating seed knowledge from KEGG, MetaCyc, and HMDB with GNN-predicted and BioTransformer-generated reaction pairs, yielding the knowledge layer that MetDNA3 uses for annotation propagation.

## When to use

Use when building or expanding a knowledge-driven metabolic reaction network from KEGG/MetaCyc/HMDB records with predicted reaction pairs, or when verifying curated MRN metabolite/reaction-pair counts.

## When NOT to use

- Input is already an annotated feature table and the goal is annotation propagation rather than network construction (use the MRN as a knowledge layer instead of re-curation)
- You require a validated FDR for the MRN itself — no FDR is reported because non-RPs are undefined under the screening conditions
- The analysis is purely statistical/clustering over features with no biochemical reaction knowledge involved

## Inputs

- Metabolite and reaction records from KEGG, MetaCyc, and HMDB (seed knowledge network)
- Candidate biotransformation pairs (Δmass-matched, with structures for overlap calculation)
- MS1 feature table, MS2 spectra (.mgf/.msp), and seed annotation table (e.g., NIST urine HILIC-MS(+) demo data) for data-constrained filtering
- GNN model and BioTransformer for reaction-pair prediction/generation

## Outputs

- Two-layer knowledge MRN as an igraph object (765,755 metabolites, 2,437,884 reaction pairs)
- Data-constrained MRN edge table (e.g., data_constrained_mrn_edges.csv; 2,993 metabolites, 55,674 reaction pairs for NIST urine HILIC-MS(+))
- Reaction-pair tables split by provenance: reported (14,974), GNN-predicted (1,432,469), BioTransformer-generated (990,441)

## How to apply

Assemble the seed knowledge network by integrating metabolite and reaction records from KEGG, MetaCyc, and HMDB (53,583 metabolites, 14,974 reported reaction pairs in this paper). Apply a two-step pre-screening strategy to candidate biotransformations — Δmass matching followed by a structural overlap threshold of ≥ 0.7 — to reduce false positives. Train and apply the graph neural network on the pre-screened candidates to add predicted reaction pairs (1,432,469 here), then run BioTransformer to generate unknown metabolites and reaction links (990,441 here), recognizing its reliance on predefined reaction types. Assemble the full two-layer network as an igraph object so it can serve as the knowledge layer for MetDNA3's annotation propagation, and optionally constrain it with modified dot-product MS2 spectral similarity (≥ 0.5 via getMatchParam/SpectraTools) to form the data-constrained MRN.

## Related tools

- **MetDNA3 (MrnAnnoAlgo3)** (Core algorithm module providing MRN construction, GNN-based reaction prediction, and igraph-based knowledge-layer assembly) — github.com/ZhuMetLab/MrnAnnoAlgo3
- **BioTransformer** (Generates unknown metabolites and their reaction links (990,441 generated reaction pairs), limited by predefined reaction types)
- **igraph** (Assembles and stores the curated two-layer MRN as a graph object)
- **SpectraTools** (Computes modified dot-product MS2 spectral similarity (via getMatchParam) to filter reaction-linked feature pairs at ≥ 0.5 into the data-constrained MRN) — https://github.com/ZhuMetLab/SpectraTools
- **KEGG / MetaCyc / HMDB** (Seed knowledge sources supplying the 53,583 metabolites and 14,974 reported reaction pairs)

## Examples

```
devtools::install_github("ZhuMetLab/MrnAnnoAlgo3"); getMetAnno(table_anno, idx_seeds, round, obj_mrn, table_ms2_network_filter, step_mrn, topN_neighbor, lib_type_mrn, thread)
```

## Evaluation signals

- The final MRN must contain exactly 765,755 metabolites and 2,437,884 reaction pairs
- Reaction-pair provenance must sum correctly: 14,974 reported + 1,432,469 GNN-predicted + 990,441 BioTransformer-generated = 2,437,884
- Candidate reaction pairs must pass both pre-screening steps: Δmass matching and structural overlap ≥ 0.7
- For NIST urine HILIC-MS(+), the data-constrained MRN must contain exactly 2,993 metabolites and 55,674 reaction pairs, matching the Source Data for Fig. 2c–d
- Retained data-constrained edges must have modified dot-product MS2 similarity score ≥ 0.5

## Limitations

- No FDR is reported for the curated MRN because non-RPs (negative examples) are undefined under the screening conditions and GNN performance metrics are inapplicable to the filtered candidate set
- GNN-based direct prediction of reaction pairs may include a considerable number of false positives, motivating the two-step pre-screening
- BioTransformer has a limited success rate and its predefined reaction types may introduce species-specific biases in generated unknown metabolites

## Evidence
