---
name: feature-cluster-aggregation-analysis
description: Use when a RAMClustR (post-XCMS) clustered object is available and the
  goal is to evaluate whether aggregating clustered MS features into compound-level
  spectra lowers quantitative analytical variation versus single-feature measurements.
when_to_use_negative:
- Input is a raw or unclustered XCMS feature table — clustering (RAMClustR) must precede
  this CV comparison
- The goal is spectral-matching-based annotation (e.g. idMS/MS spectra export for
  MSFinder or Sirius) rather than quantitative variation assessment
- The object lacks a SpecAbund matrix (e.g. only MS2 spectra were produced), leaving
  no compound-level abundance to compare
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: RAMClustR
  role: Provides the clustered ramclustobj whose SpecAbund matrix and per-feature
    intensities are compared via CV; installed as an R package from GitHub or bioconda
  repo: https://github.com/cbroeckl/RAMClustR
- name: R
  role: Analysis environment in which the CV computations, per-cluster aggregation,
    and CSV export are performed
- name: XCMS
  role: Upstream feature detection producing the aligned features that were clustered
    by RAMClustR before this analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/ac501530d
    title: 'RAMClust: A Novel Feature Clustering Method Enables Spectral-Matching-Based
      Annotation for Metabolomics Data'
  source_content_hash: sha256:bd70ba24473b2610dab84c38dfc0e87dae291dae71403f2939b5c31c279aa123
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cbroeckl/RAMClustR
  tools:
  - R
  - RAMClustR
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# feature-cluster-aggregation-analysis

## Summary

Quantifies whether RAMClustR compound-level spectra (SpecAbund) reduce analytical variation relative to their component single features by comparing per-cluster coefficient of variation (CV = SD/mean) of spectral abundances against the mean and median CVs of the constituent features.

## When to use

Use when a RAMClustR (post-XCMS) clustered object is available and the goal is to evaluate whether aggregating clustered MS features into compound-level spectra lowers quantitative analytical variation versus single-feature measurements.

## When NOT to use

- Input is a raw or unclustered XCMS feature table — clustering (RAMClustR) must precede this CV comparison
- The goal is spectral-matching-based annotation (e.g. idMS/MS spectra export for MSFinder or Sirius) rather than quantitative variation assessment
- The object lacks a SpecAbund matrix (e.g. only MS2 spectra were produced), leaving no compound-level abundance to compare

## Inputs

- RAMClustR clustered object (ramclustobj) containing the SpecAbund matrix and feature-by-sample intensity matrix
- Post-XCMS feature table / xcmsObj from a metabolomics dataset (e.g. faahKO CDF files)
- Cluster membership mapping from features to compound-level clusters

## Outputs

- cv_comparison_table.csv with one row per cluster: cluster ID, SpecAbund CV, mean feature CV, median feature CV
- Summary statistics supporting a Figure 3d-style comparison of spectral vs. feature-level variation

## How to apply

Load the RAMClustR clustered object from the prior post-XCMS clustering task (e.g. on the faahKO dataset), extracting the SpecAbund matrix and the feature-by-sample intensity matrix. For each cluster, compute the CV (standard deviation / mean) of the SpecAbund values across all samples. Independently compute the CV of each individual feature across all samples, then aggregate per cluster to obtain the mean and median feature-level CV of its component features. Assemble a one-row-per-cluster table with cluster ID, SpecAbund CV, mean feature CV, and median feature CV, and write it to cv_comparison_table.csv. Rationale: features from the same compound should share the same quantitative trend across samples, so averaging signals within a cluster suppresses feature-level noise, reproducing the paper's finding that spectral aggregation reduces quantitative analytical variation compared to single-feature measures.

## Related tools

- **RAMClustR** (Provides the clustered ramclustobj whose SpecAbund matrix and per-feature intensities are compared via CV; installed as an R package from GitHub or bioconda) — https://github.com/cbroeckl/RAMClustR
- **R** (Analysis environment in which the CV computations, per-cluster aggregation, and CSV export are performed)
- **XCMS** (Upstream feature detection producing the aligned features that were clustered by RAMClustR before this analysis)

## Examples

```
cv <- function(x) sd(x)/mean(x); spec_cv <- apply(ramclustobj$SpecAbund, 1, cv); feat_cv <- apply(ramclustobj$xcmsObj$intensity, 2, cv)
```

## Evaluation signals

- Per-cluster SpecAbund CVs are predominantly lower than both the corresponding mean and median feature CVs, reproducing the reported Figure 3d pattern
- cv_comparison_table.csv has exactly one row per cluster in the ramclustobj, with all four columns populated and no NaN CVs (or NaNs explained by zero-mean features)
- Number of clusters analyzed matches the cluster count of the input ramclustobj (e.g. ~120,000 compound measurements in the reported analysis)
- Feature-level CVs match a recomputation of sd/mean per feature column from the feature-by-sample intensity matrix

## Limitations

- Features with near-zero mean intensity across samples yield unstable/inflated CVs and can distort per-cluster mean/median feature CV
- The comparison presumes the clustering assumption that features from the same compound share approximately the same quantitative trend across samples; clusters violating this reduce the expected CV benefit
- Demonstrated on the faahKO LC-MS dataset with post-XCMS features; behavior on other platforms or feature-finding software was not validated by this analysis

## Evidence

- [abstract] reduces quantitative analytical variation compared to single-feature measures: [evidence span withheld — anchor status: not_found]
- [intro] The second is that two features derived from the same compound will have (approximately) the same quantitative trend across all samples: "The second is that two features derived from the same compound will have (approximately) the same quantitative trend across all samples in the xcms sample set."
- [methods] RAMClustR: post-XCMS Feature Clustering: "RAMClustR: post-XCMS Feature Clustering"
- [readme] A feature clustering algorithm for non-targeted mass spectrometric metabolomics data.: "A feature clustering algorithm for non-targeted mass spectrometric metabolomics data. This method is compatible with gas and liquid chromatography coupled mass spectrometry"
- [readme] print(ramclustobj$SpecAbund[,1:6]): "or inspect the `ramclustobj` for feature retention times, annotations etc.
print(ramclustobj$ann)
print(ramclustobj$nfeat)
print(ramclustobj$SpecAbund[,1:6])"
