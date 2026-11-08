---
name: spectral-similarity-nearest-neighbor-ranking
description: Use when a single differential feature (e.g. the top-prioritized globaltest
  hit in a Pleurotus eryngii 0% vs 80% olive mill solid waste contrast) must be placed
  in spectral-similarity context by extracting its nearest-neighbor edges for network
  overlay.
when_to_use_negative:
- When the goal is to re-cluster all features de novo rather than to contextualize
  one already-prioritized feature — k-medoids grouping is a separate msFeaST workflow
  step
- When only the statistical prioritization (globaltest p-values) is needed and no
  spectral-similarity network context is requested
- When only pre-processed example data must be viewed without any custom edge extraction
  — the msFeaST_Dashboard_bundle.html plus dashboard_data.json alone suffices and
  no ranking step is needed
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: msFeaST
  role: Provides the feature table, the spectral-similarity matrix, the k-medoids
    clusters, and the interactive network/dashboard on which the top-30 neighbor edges
    are overlaid
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: Statistical feature-set testing that identifies the top-prioritized differential
    feature whose neighbors are ranked
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:38d78f295b5bb4643781e78eaeb67a33dc54ed7a63a40b8b5ba309e4ed63cded
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spectral-similarity-nearest-neighbor-ranking

## Summary

Rank all features by their pairwise mass-spectral similarity to a prioritized feature of interest and export the top-N (e.g. 30) neighbors as an edge list carrying similarity scores, so connection strength can be inspected on the msFeaST network.

## When to use

Use when a single differential feature (e.g. the top-prioritized globaltest hit in a Pleurotus eryngii 0% vs 80% olive mill solid waste contrast) must be placed in spectral-similarity context by extracting its nearest-neighbor edges for network overlay.

## When NOT to use

- When the goal is to re-cluster all features de novo rather than to contextualize one already-prioritized feature — k-medoids grouping is a separate msFeaST workflow step
- When only the statistical prioritization (globaltest p-values) is needed and no spectral-similarity network context is requested
- When only pre-processed example data must be viewed without any custom edge extraction — the msFeaST_Dashboard_bundle.html plus dashboard_data.json alone suffices and no ranking step is needed

## Inputs

- Precomputed pairwise spectral-similarity matrix (feature x feature, e.g. MS2DeepScore scores) from the msFeaST example data
- msFeaST feature table / dashboard_data.json from the illustrative Pleurotus eryngii example
- Identity of the target differential feature (top-prioritized feature from globaltest testing, e.g. the Fig. 2B feature)

## Outputs

- Edge list file fig2b_top30_edges.csv with exactly 30 rows of (feature_A, feature_B, similarity)
- Network overlay showing the 30 top-neighbor edges on the msFeaST interactive dashboard network

## How to apply

Load the msFeaST illustrative-example feature table and the precomputed pairwise spectral similarity matrix (MS2DeepScore-based, as in notebooks/data/omsw_pleurotus_ms2deepscore) from github.com/kevinmildau/msFeaST. Identify the target feature as the top-prioritized feature from the globaltest-based group-wise testing, then retrieve its row in the similarity matrix and rank all other features in descending order of similarity. Take the top-30 neighbors and assemble an edge list of (feature_A, feature_B, similarity) rows — one row per edge, each retaining the pairwise score. Overlay these edges on the interactive msFeaST network (msFeaST_Dashboard_bundle.html with dashboard_data.json) to check whether the differential feature connects beyond its own small k-medoids cluster, as reported in the paper's Section 3 / Supplementary Fig. S1. The rationale is that msFeaST integrates semi-quantitative experimental information with mass-spectral structural similarity, so edges must carry similarity values, not just binary adjacency, to judge connection strength.

## Related tools

- **msFeaST** (Provides the feature table, the spectral-similarity matrix, the k-medoids clusters, and the interactive network/dashboard on which the top-30 neighbor edges are overlaid) — https://github.com/kevinmildau/msFeaST
- **globaltest** (Statistical feature-set testing that identifies the top-prioritized differential feature whose neighbors are ranked)

## Examples

```
edges = sim.loc[feat].sort_values(ascending=False).head(30); edges.to_csv('fig2b_top30_edges.csv')  # sim = pairwise spectral-similarity matrix from msFeaST example data
```

## Evaluation signals

- Output file contains exactly 30 edges, one row per (feature_A, feature_B) pair
- Every edge carries a non-null pairwise spectral-similarity score (not just adjacency)
- Similarity values in the edge list are the 30 highest values of the target feature's row in the similarity matrix (descending-order invariant)
- Visual check: edges extend beyond the target feature's own small k-medoids cluster, matching the connectivity reported in Section 3 / Supplementary Fig. S1

## Limitations

- msFeaST pipeline is tested on macOS and Linux; Windows support is untested and requires the Anaconda Prompt
- The R dependency chain (Matrix 1.6-5, survival 3.5-8, globaltest 5.56.0 via Bioconductor 3.18 with R 4.3.3) is version-pinned and conda R packaging is unreliable, so remotes/BiocManager installs are required; a faulty cached rscript path can install packages into the wrong R
- The dashboard bundle alone cannot recompute edge lists — ranking requires the installed python/R environment or the similarity matrix directly

## Evidence

- [results] workflow steps for ranking neighbors: [evidence span withheld — anchor status: not_found]
- [results] validation of edge list: [evidence span withheld — anchor status: not_found]
- [abstract] k-medoids clustering as proxy for structural similarity grouping: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [readme] example data location in repository: [evidence span withheld — anchor status: external_doc]
- [readme] example notebooks as complete pipeline: [evidence span withheld — anchor status: external_doc]
