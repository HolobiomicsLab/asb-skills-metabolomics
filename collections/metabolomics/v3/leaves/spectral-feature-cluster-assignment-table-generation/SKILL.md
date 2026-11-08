---
name: spectral-feature-cluster-assignment-table-generation
description: Use when a specXplore-style .mgf MS/MS feature set needs a tabular, multi-k
  grouping overlay derived from the ms2deepscore similarity matrix rather than network
  edge thresholds.
when_to_use_negative:
- You need hierarchically constrained medoid clusters — the authors avoided hierarchical
  clustering with medoid linkage because of lacking Python implementation and suboptimality
  at any level of k.
- You need guaranteed consistency of cluster assignments across different k settings
  — k-medoid assignments may vary across k since no cross-k agreement is enforced.
- Your feature set contains spectra/metabolites poorly covered by ms2deepscore training
  data — similarity scores may be unreliable (also beware ms2deepscore issue 199 on
  macOS arm64).
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: specXplore
  role: importing pipeline that produces the session data object and pairwise similarity
    matrix the clustering is run on
  repo: https://github.com/kevinmildau/specXplore
- name: ms2deepscore
  role: provides the pairwise spectral similarity matrix used as the input to the
    distance conversion for k-medoid clustering
- name: kmedoids
  role: Python package used to perform the k-medoid clustering over the tested k grid
- name: pandas
  role: used to hold and export the per-feature cluster label table as cluster_assignments.csv
- name: Python
  role: implementation environment for the clustering workflow inside the specXplore
    dashboard
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:9b150bcdf87e2bc9bd8dea83b15874f368af506d2fdfb35d4657b3151d2bb9e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - Python
  - kmedoids
  - ms2deepscore
  - pandas
  - specXplore
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Spectral feature cluster assignment table generation

## Summary

Generate a per-feature table of k-medoid cluster labels across multiple values of k, computed from the ms2deepscore pairwise similarity matrix that backs the specXplore dashboard, so that clustering overlays can complement the t-SNE overview.

## When to use

Use when a specXplore-style .mgf MS/MS feature set needs a tabular, multi-k grouping overlay derived from the ms2deepscore similarity matrix rather than network edge thresholds.

## When NOT to use

- You need hierarchically constrained medoid clusters — the authors avoided hierarchical clustering with medoid linkage because of lacking Python implementation and suboptimality at any level of k.
- You need guaranteed consistency of cluster assignments across different k settings — k-medoid assignments may vary across k since no cross-k agreement is enforced.
- Your feature set contains spectra/metabolites poorly covered by ms2deepscore training data — similarity scores may be unreliable (also beware ms2deepscore issue 199 on macOS arm64).

## Inputs

- specXplore session data object or precomputed ms2deepscore pairwise similarity matrix with matching feature identifiers
- .mgf file of MS/MS spectra processed by the specXplore importing pipeline (feature_id metadata key)

## Outputs

- cluster_assignments.csv: one row per MS/MS feature, one column of k-medoid cluster labels per tested k value

## How to apply

Run the specXplore importing pipeline on the .mgf spectral data (feature identifier key 'feature_id') so ms2deepscore, modified cosine, and spec2vec pairwise similarities are computed and a t-SNE embedding of the ms2deepscore matrix is generated. Convert the precomputed ms2deepscore pairwise similarity matrix into the distance representation required for partitioning. Run k-medoid clustering with the kmedoids Python package over a user-selected grid of k values, recording each feature's cluster label at every k; note the paper prescribes no automatic k-tuning guideline. Compile the assignments into a single table with one row per feature and one column of cluster labels per k (e.g. cluster_assignments.csv). Treat the output as valid if assignments are nondegenerate across the tested k range and the groupings show good agreement with t-SNE projections and topological insights, consistent with the paper's observations.

## Related tools

- **specXplore** (importing pipeline that produces the session data object and pairwise similarity matrix the clustering is run on) — https://github.com/kevinmildau/specXplore
- **ms2deepscore** (provides the pairwise spectral similarity matrix used as the input to the distance conversion for k-medoid clustering)
- **kmedoids** (Python package used to perform the k-medoid clustering over the tested k grid)
- **pandas** (used to hold and export the per-feature cluster label table as cluster_assignments.csv)
- **Python** (implementation environment for the clustering workflow inside the specXplore dashboard)

## Examples

```
conda activate specxplore_environment; jupyter-notebook notebooks/demo.ipynb  # run importing pipeline on .mgf, then kmedoids on the ms2deepscore matrix over a k grid
```

## Evaluation signals

- Every feature_id present in the .mgf/session data object appears exactly once as a row in cluster_assignments.csv (no dropped or duplicated features).
- Each tested k column contains labels forming exactly k distinct, nonempty clusters (nondegenerate assignments).
- Cluster labels align 1:1 with feature identifiers of the ms2deepscore similarity matrix (pandas index/columns match).
- Visual check: k-medoid grouping colors on the t-SNE embedding show good agreement with t-SNE projections and topological insights, as reported in the paper.
- Assignments were recomputed reproducibly from the same similarity matrix; per the paper no quantitative optimality criterion is reported, so sanity-check against the k grid chosen.

## Limitations

- No guidelines for automatic k-medoid cluster tuning are provided; stronger guidance would require further work on measuring and comparing optimality.
- Cluster assignments are not enforced to agree across different values of k and may vary between settings.
- Hierarchical clustering with medoid linkage was ruled out due to lacking Python implementation and suboptimality of hierarchically constrained clusters at any k.
- ms2deepscore works best for spectra/metabolites covered by or related to its training data; no coverage check is reported.
- On macOS arm64, ms2deepscore issue 199 can silently produce unreliable similarity predictions that feed the clustering.

## Evidence

- [results] The k-medoid clustering overlay groups spectral features directly from the pairwise similarity matrix: [evidence span withheld — anchor status: not_found]
- [results] K-medoid clustering agreement with t-SNE: "K-medoid clustering tends to show good agreement with both t-SNE projections and topological insights"
- [discussion] No automatic k tuning guideline: "More work on measuring and comparing optimality would be needed to provide users with stronger guidelines for automatic cluster tuning."
- [discussion] Hierarchical medoid clustering rejected: "due to both lacking implementation availability in Python and suboptimality of the hierarchically constrained clusters at any level of k"
- [methods] Import pipeline and similarity scores: "automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores."
- [readme] README: pipeline input format and feature key: [evidence span withheld — anchor status: external_doc]
- [readme] README: pipeline output consumed by dashboard: [evidence span withheld — anchor status: external_doc]
