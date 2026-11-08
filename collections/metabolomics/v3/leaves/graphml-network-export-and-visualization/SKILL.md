---
name: graphml-network-export-and-visualization
description: Use when you have a spin system feature similarity matrix from MADByTE
  and need to produce network graphs (full association, similarity, hybrid) for clustering
  inspection, dereplication validation, or bioactivity overlay.
when_to_use_negative:
- Your goal is compound-level spectral assignment or dereplication against a reference
  library rather than inter-sample network comparison — use MADByTE's dereplication
  module instead of network export
- Your sample set is a single mixture where inter-sample spin system matching is undefined;
  MADByTE networks require comparing features across multiple samples
- Compound classes with exclusively shielded 1H signals (sterols, low MW isoprenoids,
  lipids, volatile terpenes) are the targets — these are precluded from the TOCSY
  <2.5 ppm shielded-quadrant exclusion and will not form meaningful network nodes
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3314
tools:
- name: MADByTE
  role: generate_network module assembles the pruned spin system similarity graph
    into the three named network outputs and serializes them in graphML format
  repo: https://github.com/liningtonlab/MADByTE
- name: Gephi
  role: imports the exported .graphml files and applies the Force Atlas 2 layout algorithm
    for visualization of chemical similarity sub-clusters and bioactivity hotspots
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_007
  - task_003
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:5fca8acb76ba8c22f1581fd2233381e33b9bc1bec01f02c346c65b118ef6d9a8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  - generate_network
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# graphml-network-export-and-visualization

## Summary

Export MADByTE-generated chemical similarity networks (full association, similarity, and hybrid variants) as graphML files and visualize them in Gephi with the Force Atlas 2 layout to reveal clustering of spin system features across prefractionated extract samples.

## When to use

Use when you have a spin system feature similarity matrix from MADByTE and need to produce network graphs (full association, similarity, hybrid) for clustering inspection, dereplication validation, or bioactivity overlay.

## When NOT to use

- Your goal is compound-level spectral assignment or dereplication against a reference library rather than inter-sample network comparison — use MADByTE's dereplication module instead of network export
- Your sample set is a single mixture where inter-sample spin system matching is undefined; MADByTE networks require comparing features across multiple samples
- Compound classes with exclusively shielded 1H signals (sterols, low MW isoprenoids, lipids, volatile terpenes) are the targets — these are precluded from the TOCSY <2.5 ppm shielded-quadrant exclusion and will not form meaningful network nodes

## Inputs

- Spin system feature pairwise similarity/correlation matrix from MADByTE correlation_matrix_generation (asymmetric scores for all sample pairs)
- Sample-to-prefraction node assignment table (e.g. 85-sample prefractionated library, 9 spiked prefraction samples)
- Optional node attribute table: BioMAP antimicrobial screening results (15 clinically relevant bacterial pathogens) for the 34 active prefractions

## Outputs

- Three graphML network files: full association network (Output 1), similarity network (Output 2), hybrid network (Output 3)
- Bioactivity-annotated graphML with node attributes encoding narrow-to-broad spectrum activity for the 34 active prefractions
- Gephi Force Atlas 2 network layouts revealing sub-clusters of related chemical space (e.g. novobiocin-linked region containing 1565C/1565D, macrolide clusters, 2108D/2108E hotspot)

## How to apply

Load the spin system correlation/similarity matrix produced by correlation_matrix_generation and pass it to MADByTE's generate_network module, which creates nodes for all spin system features and connects pairs whose pairwise similarity exceeds the minimum threshold, defining each edge by the higher of the two directional similarity scores (A vs B or B vs A). Prune the graph by removing nodes with only a single connection to a spin system and dropping non-reciprocal edges. Serialize the resulting networks — the full association network, the similarity network, and the hybrid network — each as a .graphml file. Import the graphML into Gephi and apply the Force Atlas 2 layout algorithm to expose sub-clusters, e.g. spiked prefractions clustering around novobiocin, mupirocin, and erythromycin. For bioactivity overlay, attach BioMAP screening results (15 pathogens) as node attributes on the 34 active prefraction nodes before export so hotspot clusters like 2108D/2108E are visible and machine-readable.

## Related tools

- **MADByTE** (generate_network module assembles the pruned spin system similarity graph into the three named network outputs and serializes them in graphML format) — https://github.com/liningtonlab/MADByTE
- **Gephi** (imports the exported .graphml files and applies the Force Atlas 2 layout algorithm for visualization of chemical similarity sub-clusters and bioactivity hotspots)

## Examples

```
conda activate madbyte; python madbyte_gui.py  # run generate_network, then export .graphml and open in Gephi with Force Atlas 2
```

## Evaluation signals

- Each of the three .graphml output files (full association, similarity, hybrid) exists, parses as a valid graphML graph, and its nodes/edge weights correspond to entries in the input similarity matrix
- In the spiked 9-sample network, samples containing each spiked compound (novobiocin, mupirocin, erythromycin) form distinct sub-clusters connected to their reference compound by at least two shared spin system features
- The 85-sample hybrid network separates samples into distinct regions of related chemical space and places prefractions 1565C and 1565D in the novobiocin-linked region, matching the published Figure 4/5 layouts
- The bioactivity-annotated graphML contains exactly the 34 BioMAP-active prefraction nodes with bioactivity attributes and includes the 2108D/2108E hotspot as in Figure 6A
- Pruned networks contain no nodes with only a single connection to a spin system and no non-reciprocal edges

## Limitations

- MADByTE will not identify every spin system in a given mixture, so network completeness is limited by upstream feature detection
- Peak-picking algorithms perform poorly in the crowded shielded region (<2.5 ppm), and TOCSY cross-peaks with both F1 and F2 < 2.5 ppm are excluded, so networks systematically lack nodes from shielded regions
- Node pruning (single-connection removal) and non-reciprocal edge removal deliberately discard low-confidence features, so exported networks are not exhaustive representations of the similarity matrix

## Evidence

- [readme] README GUI launch: [evidence span withheld — anchor status: not_found]
