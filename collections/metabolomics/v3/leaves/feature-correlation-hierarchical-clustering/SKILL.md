---
name: feature-correlation-hierarchical-clustering
description: Use when you have a table of statistically significant features (e.g.
  permutation p-value < 0.01 from MB-VIP testing) and need to group them into coherent
  feature sets based on how their intensities co-vary across samples.
when_to_use_negative:
- Input is a raw unfiltered intensity matrix where no significance filter (e.g. permutation
  p-value < 0.01) has yet been applied — first run mb_vip_permtest to select significant
  features
- Goal is structural clustering based on retention time and m/z adduct/isotopologue
  patterns rather than co-variance of intensities — use the get_structural_clusters
  step instead
- Data are single-assay only where no multi-assay integration or cross-assay linking
  is intended; MAMSI is designed for multi-assay LC-MS metabolomics
tools:
- name: mamsi
  role: MamsiStructSearch.get_correlation_clusters performs the Pearson correlation
    computation, dissimilarity conversion, agglomerative clustering, and silhouette-based
    dendrogram flattening
  repo: https://github.com/kopeckylukas/py-mamsi
- name: scikit-learn
  role: Supports agglomerative hierarchical clustering and silhouette-score computation
    used under the hood
- name: scipy
  role: Linkage computation (complete linkage) for the hierarchical dendrogram
- name: pandas
  role: Holds the significant-feature intensity table and column metadata for clustering
- name: matplotlib
  role: Renders the correlation heatmap and silhouette diagnostic plots
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:32e1c2f9a2f1e30d81fe13fd3f144d27cb75bc433631d09e7d15a77015e5977e
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# feature-correlation-hierarchical-clustering

## Summary

Clusters statistically significant LC-MS features by their intensity correlation across samples using agglomerative hierarchical clustering with complete linkage on a |1−rὢ| dissimilarity matrix, with the cluster count chosen by the maximum mean silhouette score.

## When to use

Use when you have a table of statistically significant features (e.g. permutation p-value < 0.01 from MB-VIP testing) and need to group them into coherent feature sets based on how their intensities co-vary across samples.

## When NOT to use

- Input is a raw unfiltered intensity matrix where no significance filter (e.g. permutation p-value < 0.01) has yet been applied — first run mb_vip_permtest to select significant features
- Goal is structural clustering based on retention time and m/z adduct/isotopologue patterns rather than co-variance of intensities — use the get_structural_clusters step instead
- Data are single-assay only where no multi-assay integration or cross-assay linking is intended; MAMSI is designed for multi-assay LC-MS metabolomics

## Inputs

- Table of statistically significant LC-MS features (features with permutation p-value < 0.01 from mb_vip_permtest), rows as samples and columns as features
- Column headers in the format AssayName_RTsec_m/z (e.g. HPOS_, LPOS_, LNEG_ prefixed feature names)

## Outputs

- Feature-to-correlation-cluster assignment table
- Best cluster count k and its mean silhouette score
- correlation_heatmap.png (pairwise correlation diagnostic heatmap)
- silhouette_plot.png (silhouette diagnostic plot)

## How to apply

Take the significant-feature table (features with permutation p-value < 0.01 from the mb_vip_permtest step) with samples as rows and features as columns, and compute the pairwise Pearson correlation matrix across samples using mamsi's MamsiStructSearch.get_correlation_clusters (cor_method='pearson'). Convert the correlation matrix into a dissimilarity matrix via d_ij = |1 − r_ij|, then run agglomerative hierarchical clustering with complete (farthest point) linkage (linkage_method='complete', metric='euclidean'). Flatten the dendrogram using the silhouette criterion (flat_method='silhouette') while searching up to max_clusters=11 clusters; record the best k and its mean silhouette score. Emit the feature-to-cluster assignments plus diagnostic plots (correlation_heatmap.png, silhouette_plot.png). The tutorial reference run on the AddNeuroMed Alzheimer's sample data resolved to 8 clusters with a mean silhouette score of ~0.244.

## Related tools

- **mamsi** (MamsiStructSearch.get_correlation_clusters performs the Pearson correlation computation, dissimilarity conversion, agglomerative clustering, and silhouette-based dendrogram flattening) — https://github.com/kopeckylukas/py-mamsi
- **scikit-learn** (Supports agglomerative hierarchical clustering and silhouette-score computation used under the hood)
- **scipy** (Linkage computation (complete linkage) for the hierarchical dendrogram)
- **pandas** (Holds the significant-feature intensity table and column metadata for clustering)
- **matplotlib** (Renders the correlation heatmap and silhouette diagnostic plots)

## Examples

```
struct.get_correlation_clusters(flat_method='silhouette', max_clusters=11)
```

## Evaluation signals

- The chosen number of clusters is the k with the maximum mean silhouette score within the searched range (max_clusters=11)
- Every significant feature receives exactly one cluster label
- The tutorial reference run resolves to 8 clusters with a mean silhouette score of ~0.244
- Both correlation_heatmap.png and silhouette_plot.png are produced as diagnostic outputs
- The feature-to-cluster assignment table covers every feature in the input significant-feature table with no gaps

## Limitations

- Requires a prior significance filter (permutation p-value < 0.01) — clustering raw unfiltered features is out of scope
- Correlation structure alone does not imply structural relation; structural links (isotopologues, adducts, cross-assay) must be assessed separately via get_structural_clusters
- Cluster count search is capped at max_clusters=11 by default; if the true cluster count exceeds this, the result will be suboptimal
- Tested on metabolomics phenotyping data from the National Phenome Centre's LC-MS assays; applicability to other LC-MS data types is expected but not guaranteed by the authors

## Evidence

- [methods] Significant features are clustered by computing a dissimilarity matrix from intensity correlations, then applying agglomerative hierarchical clustering with complete (farthest point) linkage, choosing the number of clusters by the highest mean silhouette score.: [evidence span withheld — anchor status: not_found]
- [readme] Correlation clustering via silhouette selection: "struct.get_correlation_clusters(flat_method='silhouette', max_clusters=11)"
- [readme] Correlation clustering operates on significant features only: "mask = np.where(p_vals < 0.01)
selected = x.iloc[:, mask[0]]"
