---
name: silhouette-based-optimal-cluster-count-selection
description: Use when a table of statistically significant features (e.g. MB-VIP permutation
  p-value < 0.01) must be partitioned into correlation-based clusters and the cluster
  count k is unknown and must be chosen objectively.
when_to_use_negative:
- Features are already grouped by structural properties (RT windows, isotopologue/adduct
  signatures) via MamsiStructSearch.get_structural_clusters — correlation clustering
  is a complementary, not replacement, grouping.
- The dataset contains fewer informative features or samples than needed to support
  up to 11 candidate cluster counts; silhouette scores over too few samples will be
  unstable.
- The goal is a fixed, predefined number of clusters (k known a priori) rather than
  objective selection of k.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: mamsi
  role: Provides MamsiStructSearch.get_correlation_clusters, which computes correlation
    clusters and flattens the hierarchy by silhouette
  repo: https://github.com/kopeckylukas/py-mamsi
- name: scipy
  role: Supplies agglomerative hierarchical clustering with complete (farthest point)
    linkage on the correlation dissimilarity matrix
- name: scikit-learn
  role: Supplies silhouette scoring used to select the optimal cluster count
- name: seaborn
  role: Dependency used for the correlation heatmap diagnostic
- name: pandas
  role: Holds the significant-feature table (samples × features) fed to correlation
    clustering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:2fdb77b5beaca84c4346117976dfd44dcbd749b66c69194df6492d523e853c43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - Python
  - mamsi
  - matplotlib
  - numpy
  - pandas
  - scikit-learn
  - scipy
  - seaborn
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# silhouette-based-optimal-cluster-count-selection

## Summary

Selects the number of clusters for hierarchical clustering of significant LC-MS features by flattening a complete-linkage dendrogram with the mean silhouette criterion, as implemented in MAMSI's MamsiStructSearch.get_correlation_clusters. Features are grouped by |1−r_ij| correlation-based dissimilarity rather than m/z/RT alone.

## When to use

Use when a table of statistically significant features (e.g. MB-VIP permutation p-value < 0.01) must be partitioned into correlation-based clusters and the cluster count k is unknown and must be chosen objectively.

## When NOT to use

- Features are already grouped by structural properties (RT windows, isotopologue/adduct signatures) via MamsiStructSearch.get_structural_clusters — correlation clustering is a complementary, not replacement, grouping.
- The dataset contains fewer informative features or samples than needed to support up to 11 candidate cluster counts; silhouette scores over too few samples will be unstable.
- The goal is a fixed, predefined number of clusters (k known a priori) rather than objective selection of k.

## Inputs

- Feature table of significant LC-MS features (rows = samples, columns = features, e.g. the selected = x.iloc[:, mask[0]] output where p_vals < 0.01 from mb_vip_permtest)
- cor_method for correlation estimation (default 'pearson')
- linkage_method ('complete') and metric ('euclidean') for agglomerative clustering
- flat_method ('silhouette') and max_clusters (11) for dendrogram flattening

## Outputs

- Feature-to-cluster assignment table (one cluster label per significant feature)
- Best number of clusters k and its mean silhouette score
- correlation_heatmap.png diagnostic heatmap
- silhouette_plot.png diagnostic plot

## How to apply

Compute the pairwise Pearson correlation matrix across samples for the significant feature columns and convert it to a dissimilarity matrix d_ij = |1 − r_ij|. Run agglomerative hierarchical clustering with complete (farthest point) linkage using scipy/sklearn on this dissimilarity matrix. Instead of picking k arbitrarily, flatten the dendrogram with the silhouette criterion (flat_method='silhouette'), searching candidate k values up to max_clusters=11, and keep the k with the highest mean silhouette score. In the MAMSI tutorial reference run this procedure resolved to 8 clusters with a mean silhouette score of ~0.244. Emit per-feature cluster labels plus the correlation heatmap and silhouette plot diagnostics (correlation_heatmap.png, silhouette_plot.png) so the chosen k can be reviewed alongside structural m/z/RT-defined groups.

## Related tools

- **mamsi** (Provides MamsiStructSearch.get_correlation_clusters, which computes correlation clusters and flattens the hierarchy by silhouette) — https://github.com/kopeckylukas/py-mamsi
- **scipy** (Supplies agglomerative hierarchical clustering with complete (farthest point) linkage on the correlation dissimilarity matrix)
- **scikit-learn** (Supplies silhouette scoring used to select the optimal cluster count)
- **seaborn** (Dependency used for the correlation heatmap diagnostic)
- **pandas** (Holds the significant-feature table (samples × features) fed to correlation clustering)

## Examples

```
from mamsi.mamsi_struct_search import MamsiStructSearch; struct = MamsiStructSearch(rt_win=5, ppm=10); struct.load_lcms(selected); struct.get_correlation_clusters(flat_method='silhouette', max_clusters=11)
```

## Evaluation signals

- The selected k is the candidate (up to max_clusters=11) with the maximum mean silhouette score; the reference tutorial run resolved to 8 clusters with mean silhouette ~0.244.
- Every significant feature from the p < 0.01 MB-VIP permutation test receives exactly one cluster label — no unassigned features, no duplicate labels per feature.
- Silhouette plot (silhouette_plot.png) shows per-cluster silhouette values consistent with the reported mean score.
- Correlation heatmap (correlation_heatmap.png) exhibits block structure consistent with the returned cluster assignments.
- Cluster memberships can be cross-checked against m/z/RT structural clusters from get_structural_clusters for biological plausibility.

## Limitations

- Cluster count depends on the max_clusters search cap (11 in the tutorial); the true optimum may lie beyond the cap.
- Mean silhouette of ~0.244 in the reference run indicates modest cluster separation; correlation-based clusters may not match structurally defined (m/z/RT) groups.
- Framework was tested on metabolomics phenotyping data but is expected to generalize to other LC-MS data types; correlation structure depends on adequate sample size across assays (e.g. HPOS/LPOS/LNEG blocks).

## Evidence

- [readme] README quickstart call: [evidence span withheld — anchor status: external_doc]
- [readme] README parameter settings: "struct.get_correlation_clusters(flat_method='silhouette', max_clusters=11)"
- [abstract] abstract clustering rationale: "Then we cluster the selected predictors and compare them to groups defined by their structural properties based on retention time and mass-to-charge ratio."
- [intro] intro clustering scope: "linking statistically significant features of untargeted multi-assay liquid chromatography – mass spectrometry (LC-MS) metabolomics datasets into clusters defined by their structural properties based"
- [methods] methods feature selection upstream: "mask = np.where(p_vals < 0.01)
selected = x.iloc[:, mask[0]]"
