---
name: spin-system-edge-reciprocity-filtering
description: Use when converting a MADByTE pairwise spin-system similarity matrix
  (task_002 output) into network graphs and needing to prune asymmetric or weakly
  connected nodes/edges before graphML export.
when_to_use_negative:
- Input is a single-sample dataset with no cross-sample feature comparison — there
  are no pairwise spin-system matches to make reciprocal
- The compound class of interest contains exclusively shielded 1H signals (e.g. sterols,
  low MW isoprenoids, lipids, volatile terpenes), which MADByTE excludes from analysis
  in the shielded quadrant (<2.5 ppm)
- A feature table or already-filtered network is provided — reciprocity pruning operates
  on the raw asymmetric similarity matrix, not on post-processed graphs
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MADByTE
  role: Provides the generate_network module that builds the full association, similarity,
    and hybrid networks from the similarity matrix and applies reciprocity/single-connection
    pruning
  repo: https://github.com/liningtonlab/MADByTE
- name: Gephi
  role: Receives the exported .graphML network files for visualization using the Force
    Atlas 2 algorithm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:2f18fa799efbaa6ef993ed31da955a4fa314e57120ec10f8cd87fbc386dc7c9a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Spin-system edge reciprocity filtering

## Summary

Prunes MADByTE chemical similarity networks by removing spin-system nodes with only a single connection and dropping non-reciprocal edges, so that only mutually supported feature matches survive before the full association, similarity, and hybrid networks are serialized as graphML for Gephi visualization.

## When to use

Use when converting a MADByTE pairwise spin-system similarity matrix (task_002 output) into network graphs and needing to prune asymmetric or weakly connected nodes/edges before graphML export.

## When NOT to use

- Input is a single-sample dataset with no cross-sample feature comparison — there are no pairwise spin-system matches to make reciprocal
- The compound class of interest contains exclusively shielded 1H signals (e.g. sterols, low MW isoprenoids, lipids, volatile terpenes), which MADByTE excludes from analysis in the shielded quadrant (<2.5 ppm)
- A feature table or already-filtered network is provided — reciprocity pruning operates on the raw asymmetric similarity matrix, not on post-processed graphs

## Inputs

- Spin system correlation/similarity matrix from MADByTE correlation_matrix_generation (task_002)
- Spin system feature tables derived from combined TOCSY and HSQC spectra

## Outputs

- Full association network (.graphml)
- Similarity network (.graphml)
- Hybrid network (.graphml)

## How to apply

Load the spin system correlation/similarity matrix produced by the upstream correlation_matrix_generation step through the MADByTE package. Convert pairwise similarity scores into network edges, defining each edge between spin system nodes by the higher of the two directional similarity scores (A vs B or B vs A). Prune the graph by removing nodes containing only a single connection to a spin system and dropping edges between nodes that are not reciprocal. Run the generate_network module to build the three named network variants (full association, similarity, hybrid) and serialize each in graphML format for downstream processing in Gephi with the Force Atlas 2 layout algorithm.

## Related tools

- **MADByTE** (Provides the generate_network module that builds the full association, similarity, and hybrid networks from the similarity matrix and applies reciprocity/single-connection pruning) — https://github.com/liningtonlab/MADByTE
- **Gephi** (Receives the exported .graphML network files for visualization using the Force Atlas 2 algorithm)

## Examples

```
conda activate madbyte; python madbyte_gui.py
```

## Evaluation signals

- Each of the three .graphml output files exists and parses as a valid graphML graph
- Every retained edge between two spin-system nodes is reciprocal (present in both directional comparisons A vs B and B vs A)
- No node in the pruned graph has fewer than two connections; single-connection nodes are absent
- Edge weights in the exported graphs correspond to entries in the input similarity matrix, using the higher of the two directional scores
- The three named outputs (Output 1 full association, Output 2 similarity, Output 3 hybrid) are all produced

## Limitations

- TOCSY cross-peaks where both F1 and F2 chemical shifts are < 2.5 ppm are excluded, so networks lack nodes from the crowded shielded region
- MADByTE will not identify every spin system in a given mixture, so networks are incomplete by design
- Peak-picking algorithms perform poorly in the crowded shielded region, which propagates into feature matching and edge construction

## Evidence
