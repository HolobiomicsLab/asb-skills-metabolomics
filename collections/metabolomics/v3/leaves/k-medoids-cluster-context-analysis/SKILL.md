---
name: k-medoids-cluster-context-analysis
description: Use when a differential feature flagged by globaltest-based statistical
  testing of msFeaST feature groups needs its connectivity context quantified — e.
when_to_use_negative:
- When the goal is the statistical prioritization itself (globaltest feature-group
  testing) rather than post-hoc neighborhood/context analysis of an already-prioritized
  feature
- When only a quantification table is available and no pairwise spectral similarity
  matrix has been computed — the msFeaST preprocessing/similarity step must run first
- When plain adjacency (connectivity without per-edge similarity strength) is sufficient
  and per-edge pairwise scores are not needed
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_2269
tools:
- name: msFeaST
  role: Provides the k-medoids spectral-similarity feature grouping, the pairwise
    similarity matrix, and the interactive dashboard used to visualize the differential
    feature's top-N neighbor edges
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: Performs the feature group-wise statistical testing that identifies the top-prioritized
    differential feature whose neighborhood is then analyzed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:fccf1a213ddd0c8482f7035be1b512a7be3cde0184f7364758b3abc7707419d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# k-medoids-cluster-context-analysis

## Summary

Characterizes the spectral-similarity neighborhood of a prioritized LC-MS/MS feature beyond its k-medoids cluster by extracting top-N neighbor edges (with pairwise similarity scores) from the msFeaST similarity matrix, enabling assessment of how strongly a differential feature connects outside its own spectral cluster.

## When to use

Use when a differential feature flagged by globaltest-based statistical testing of msFeaST feature groups needs its connectivity context quantified — e.g. building a top-30 neighbor edge list from a precomputed pairwise spectral similarity matrix to check whether the feature links beyond its own cluster.

## When NOT to use

- When the goal is the statistical prioritization itself (globaltest feature-group testing) rather than post-hoc neighborhood/context analysis of an already-prioritized feature
- When only a quantification table is available and no pairwise spectral similarity matrix has been computed — the msFeaST preprocessing/similarity step must run first
- When plain adjacency (connectivity without per-edge similarity strength) is sufficient and per-edge pairwise scores are not needed

## Inputs

- msFeaST feature quantification table (e.g. dashboard_data.json from the omsw_pleurotus_ms2deepscore example)
- Precomputed pairwise spectral similarity matrix (e.g. MS2Deepscore-based) over all LC-MS/MS features
- Identifier of the target differential feature (top-prioritized feature from globaltest testing of the k-medoids feature groups)
- Neighbor count N (e.g. top-30 neighbors as in Fig. 2B)

## Outputs

- Edge list (e.g. fig2b_top30_edges.csv) with exactly N rows, each a (feature_A, feature_B, pairwise similarity) record
- Verified network-context assessment of whether the differential feature's connectivity extends beyond its k-medoids cluster

## How to apply

Load the msFeaST feature table and the precomputed pairwise spectral similarity matrix (e.g. from the illustrative Pleurotus eryngii 0% vs 80% olive mill solid waste example data in notebooks/data/omsw_pleurotus_ms2deepscore). Identify the top-prioritized differential feature from the globaltest-based testing of the k-medoids spectral-similarity clusters, then retrieve its similarity row, rank all pairwise values in descending order, and select the top-30 nearest neighbors as (feature_A, feature_B, similarity) edge records. Overlay these edges onto the interactive msFeaST dashboard network (msFeaST_Dashboard_bundle.html with dashboard_data.json) to verify whether connectivity extends beyond the small highlighted feature cluster, as reported for Fig. 2B / Supplementary Fig. S1. The rationale: k-medoids clusters are only a proxy for structurally similar features, so per-edge similarity scores must be carried through so connection strength between features can be assessed, not just adjacency.

## Related tools

- **msFeaST** (Provides the k-medoids spectral-similarity feature grouping, the pairwise similarity matrix, and the interactive dashboard used to visualize the differential feature's top-N neighbor edges) — https://github.com/kevinmildau/msFeaST
- **globaltest** (Performs the feature group-wise statistical testing that identifies the top-prioritized differential feature whose neighborhood is then analyzed)

## Examples

```
top30 = sim.loc[feat].sort_values(ascending=False).head(30); top30.to_frame(name='similarity').assign(feature_B=top30.index).to_csv('fig2b_top30_edges.csv')
```

## Evaluation signals

- Output edge list contains exactly N edges (30 for the Fig. 2B reproduction)
- Every edge row carries a pairwise similarity score drawn from the precomputed similarity matrix
- Connectivity pattern of the differential feature extends beyond its k-medoids cluster, consistent with Section 3 / Supplementary Fig. S1
- Edge similarity values match the corresponding entries of the spectral similarity matrix (row-lookup consistency check)
- Overlay onto the msFeaST dashboard network reproduces the strong out-of-cluster connectivity reported for Fig. 2B

## Limitations

- k-medoids clusters serve only as a proxy for grouping structurally similar features, so in-cluster membership does not guarantee structural identity
- msFeaST pipeline has been tested on macOS and expected to work on Linux; Windows support is untested/in progress and requires ANACONDA PROMPT
- R path caching issues can misdirect rscript package installations outside the conda environment, breaking the R dependencies (including globaltest) the analysis depends on

## Evidence

- [abstract] k-medoids clusters as proxy for structural similarity: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [other] Fig. 2B top-30 neighbor finding: [evidence span withheld — anchor status: not_found]
- [readme] dashboard example data location: [evidence span withheld — anchor status: external_doc]
