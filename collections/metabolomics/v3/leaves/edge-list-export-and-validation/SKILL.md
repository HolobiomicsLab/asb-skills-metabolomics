---
name: edge-list-export-and-validation
description: 'Use when a top-prioritized feature from globaltest-based msFeaST testing
  must be interrogated in network context: extract its k-nearest spectral neighbors
  as a weighted edge list and verify cross-cluster connectivity.'
when_to_use_negative:
- When the feature has no precomputed spectral similarity matrix available — the skill
  requires existing pairwise ms2deepscore scores, not on-the-fly similarity computation.
- When the goal is bulk network export of all features rather than neighbor analysis
  of a single prioritized differential feature.
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: msFeaST
  role: Provides the precomputed pairwise spectral similarity matrix, the k-medoids
    feature grouping, the interactive dashboard network for edge overlay, and the
    illustrative Pleurotus eryngii example data used to build and validate the edge
    list.
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: Statistical feature-set testing that prioritizes the differential feature
    whose neighbor edge list is exported.
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:dbce1a411d7560aed6fe793fbdd219092382ea3ebc19259ab53614bbbd01fa6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# edge-list-export-and-validation

## Summary

Exports a spectral-similarity edge list for a prioritized differential feature (feature_A, feature_B, similarity per row) from the msFeaST precomputed pairwise similarity matrix and validates that network connectivity extends beyond the feature's own k-medoids cluster.

## When to use

Use when a top-prioritized feature from globaltest-based msFeaST testing must be interrogated in network context: extract its k-nearest spectral neighbors as a weighted edge list and verify cross-cluster connectivity.

## When NOT to use

- When the feature has no precomputed spectral similarity matrix available — the skill requires existing pairwise ms2deepscore scores, not on-the-fly similarity computation.
- When the goal is bulk network export of all features rather than neighbor analysis of a single prioritized differential feature.

## Inputs

- msFeaST illustrative-example feature table (dashboard_data.json from notebooks/data/omsw_pleurotus_ms2deepscore/)
- Precomputed pairwise spectral similarity matrix (ms2deepscore scores) from the msFeaST repository example data
- Identity of the top-prioritized differential feature from globaltest-based testing of k-medoids feature groups

## Outputs

- CSV edge list fig2b_top30_edges.csv with exactly 30 rows, each containing feature_A, feature_B, and a pairwise similarity score
- Visual confirmation (overlay) that the differential feature's connectivity extends beyond its own k-medoids cluster

## How to apply

Identify the differential feature highlighted in the msFeaST illustrative example (Pleurotus eryngii 0% vs 80% olive mill solid waste contrast), i.e. the top-prioritized feature from the globaltest-based statistical testing of k-medoids feature groups. Retrieve similarity values between this feature and all other features from the precomputed pairwise spectral similarity matrix (ms2deepscore-based in the example) and rank them in descending order. Select the top-30 nearest neighbors and write a CSV edge list with one (feature_A, feature_B, similarity) record per edge. Overlay the edges onto the msFeaST interactive-dashboard network (dashboard_data.json from notebooks/data/omsw_pleurotus_ms2deepscore/) to confirm connectivity extends beyond the feature's small highlighted cluster, as in Supplementary Fig. S1.

## Related tools

- **msFeaST** (Provides the precomputed pairwise spectral similarity matrix, the k-medoids feature grouping, the interactive dashboard network for edge overlay, and the illustrative Pleurotus eryngii example data used to build and validate the edge list.) — https://github.com/kevinmildau/msFeaST
- **globaltest** (Statistical feature-set testing that prioritizes the differential feature whose neighbor edge list is exported.) — https://github.com/kevinmildau/msFeaST

## Examples

```
rscript -e "BiocManager::install('globaltest', version='3.18')"
```

## Evaluation signals

- Output CSV contains exactly 30 edges.
- Every edge row carries a pairwise similarity score (not null/empty).
- Connectivity of the differential feature in the overlay extends beyond its own small k-medoids cluster, consistent with the reported finding.
- Edges connect the target feature to its true top-30 neighbors as ranked by descending similarity from the precomputed matrix.

## Limitations

- Connectivity beyond the cluster is assessed visually on the dashboard network; the skill does not define a numeric connectivity threshold.
- The illustrative example is specific to the Pleurotus eryngii 0% vs 80% olive mill solid waste contrast; generalization to other datasets requires the same precomputed similarity-matrix structure.
- R package environment (globaltest via BiocManager) is sensitive to rscript path caching; the README warns that a mis-cached R path will prevent the pipeline from running successfully.

## Evidence

- [results] the displayed edges encode both connectivity and pairwise similarity, enabling assessment of connection strength between connected features: [evidence span withheld — anchor status: not_found]
- [abstract] Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package: "Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package"
- [readme] The interactive visualization dashboard works regardless of os on desktop browsers: [evidence span withheld — anchor status: external_doc]
