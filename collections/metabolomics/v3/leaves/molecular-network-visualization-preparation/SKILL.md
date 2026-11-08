---
name: molecular-network-visualization-preparation
description: Use when a spin system correlation/similarity matrix from MADByTE's correlation_matrix_generation
  step must be converted into network graph edges, pruned, and exported as graphML
  for Gephi visualization.
when_to_use_negative:
- Input is raw TOCSY/HSQC spectra or peak lists that have not yet been processed into
  a pairwise similarity matrix — the upstream correlation_matrix_generation step must
  run first.
- Input is already a finished graphML network that only needs layout/visualization
  (use Gephi directly with Force Atlas 2 rather than regenerating networks).
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: MADByTE
  role: Runs the generate_network module that converts similarity matrix values into
    the full association, similarity, and hybrid network variants
  repo: https://github.com/liningtonlab/MADByTE
- name: Gephi
  role: Consumes the exported graphML files for visualization using the Force Atlas
    2 layout algorithm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:a42314ed1276299d6868d8bd442e6f6271f9dc4caf80e4f2de91bfde495e846f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# molecular-network-visualization-preparation

## Summary

Converts the MADByTE spin system pairwise similarity matrix into three pruned network graphs (full association, similarity, hybrid) and serializes them as graphML files ready for Force Atlas 2 visualization in Gephi.

## When to use

Use when a spin system correlation/similarity matrix from MADByTE's correlation_matrix_generation step must be converted into network graph edges, pruned, and exported as graphML for Gephi visualization.

## When NOT to use

- Input is raw TOCSY/HSQC spectra or peak lists that have not yet been processed into a pairwise similarity matrix — the upstream correlation_matrix_generation step must run first.
- Input is already a finished graphML network that only needs layout/visualization (use Gephi directly with Force Atlas 2 rather than regenerating networks).

## Inputs

- Pairwise spin system similarity/correlation matrix produced by MADByTE correlation_matrix_generation (task_002)
- Spin system feature tables (TOCSY-derived nodes annotated with 13C shifts from HSQC)

## Outputs

- Full association network graph (.graphml)
- Similarity network graph (.graphml)
- Hybrid network graph (.graphml)

## How to apply

Load the spin system similarity matrix produced upstream by the correlation_matrix_generation step through the MADByTE package. Convert pairwise similarity scores into edges between spin system feature nodes, defining each edge by the higher of the two directional scores (A vs B or B vs A), since the similarity matrix is asymmetric. Prune the graph by removing nodes containing only a single connection to a spin system and dropping edges that are not reciprocal between nodes. Run the generate_network module to build the three named network variants: the full association network, the similarity network, and the hybrid network. Serialize each network in graphML format for downstream visualization in Gephi using the Force Atlas 2 layout algorithm.

## Related tools

- **MADByTE** (Runs the generate_network module that converts similarity matrix values into the full association, similarity, and hybrid network variants) — https://github.com/liningtonlab/MADByTE
- **Gephi** (Consumes the exported graphML files for visualization using the Force Atlas 2 layout algorithm)

## Examples

```
conda activate madbyte; python madbyte_gui.py
```

## Evaluation signals

- Each of the three .graphml output files exists and parses as a valid graphML graph.
- Nodes and edge weights in each output correspond to the input similarity matrix entries.
- Every edge weight equals the higher of the two directional similarity scores (A vs B or B vs A).
- No nodes with only a single connection to a spin system and no non-reciprocal edges remain after pruning.
- GraphML files load into Gephi and can be laid out with Force Atlas 2 without schema errors.

## Limitations

- MADByTE will not identify every spin system in a given mixture, so the network covers only detected features.
- In cases of spectral overlap yielding multiple candidate 13C chemical shifts, only the closest HSQC resonance value is included, which may affect node annotation.
- Peak picking performs poorly in the crowded shielded region, which can propagate missing or noisy nodes into the networks.

## Evidence
