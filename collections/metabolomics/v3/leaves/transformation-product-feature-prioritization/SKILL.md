---
name: transformation-product-feature-prioritization
description: Use when you have a pre-processed HRMS feature table (e.g., Agilent .csv
  export with blank/control-subtracted features) across a time-course or dilution
  series, and you need to prioritize features — such as transformation products —
  that share similar abundance trends without supervised labels.
when_to_use_negative:
- Input is raw .mzML data that still requires feature extraction — run mssmain.get_scans
  and mssmain.peak_pick (with peak width, replicated peak, and background subtraction
  filters) first to obtain a feature table
- The dataset lacks time-course/dilution structure, so feature abundance trends carry
  no grouping signal for DBSCAN
- You need isotope/adduct/in-source-fragment feature grouping — the article lists
  this as future work, not an implemented capability of this skill
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3473
tools:
- name: Mass-Suite (MSS)
  role: Provides dm.ms_cluster (Z-score normalization + DBSCAN/OPTICS clustering)
    and eps_assess for knee-plot eps selection; writes cluster assignments and optional
    cluster reports
  repo: https://github.com/XiminHu/mass-suite
- name: Python
  role: Execution environment for the mss package and its Pandas/Numpy/Scikit-learn
    dependencies
  repo: https://github.com/XiminHu/mass-suite
- name: scikit-learn
  role: Backend implementation of the DBSCAN clustering algorithm invoked through
    dm.ms_cluster
  repo: https://scikit-learn.org/stable/modules/generated/sklearn.cluster.DBSCAN.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:b63dbf1414d7d7c088d13bb8143d1a4b7adf3f43d2f2de11ffab56944b94eb3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# transformation-product-feature-prioritization

## Summary

Prioritizes candidate transformation-product features from HRMS time-course feature tables by unsupervised clustering of abundance trends with MSS's dm.ms_cluster (DBSCAN), grouping features whose normalized intensities co-vary across time points and flagging outliers for exclusion.

## When to use

Use when you have a pre-processed HRMS feature table (e.g., Agilent .csv export with blank/control-subtracted features) across a time-course or dilution series, and you need to prioritize features — such as transformation products — that share similar abundance trends without supervised labels.

## When NOT to use

- Input is raw .mzML data that still requires feature extraction — run mssmain.get_scans and mssmain.peak_pick (with peak width, replicated peak, and background subtraction filters) first to obtain a feature table
- The dataset lacks time-course/dilution structure, so feature abundance trends carry no grouping signal for DBSCAN
- You need isotope/adduct/in-source-fragment feature grouping — the article lists this as future work, not an implemented capability of this skill

## Inputs

- Agilent .csv feature table exported from HRMS NTA pre-processing (blank/control-subtracted features; e.g., 136 features from the dienogest/drospirenone time-course at 0, 4, 10, and 29 h)
- Feature-by-sample abundance matrix with time-course structure (formats supported by Pandas)

## Outputs

- cluster_assignments.csv with per-feature cluster labels (outliers labeled as noise)
- Optional cluster report and visualization (PCA or T-SNE dimension-reduced plots)
- Prioritized list of candidate transformation-product features (e.g., 18 features in the article's validation)

## How to apply

Load the exported .csv feature table (e.g., the dienogest/drospirenone biotransformation time-course at 0, 4, 10, and 29 h, 136 features after blank and control subtraction) and normalize feature abundances with Z-score normalization z=(x-μ)/σ, which is dm.ms_cluster's default, to eliminate skewness and kurtosis. Determine the DBSCAN eps parameter using MSS's eps_assess knee plot and set min_samples based on the feature number (the article's example uses eps=0.3 and min_samples=5). Run dm.ms_cluster with the DBSCAN algorithm to group features with similar abundance trends and exclude outliers as noise (label -1); optionally apply PCA or T-SNE dimension reduction and generate a cluster report/visualization to inspect cluster structure. Write per-feature cluster assignments (cluster_assignments.csv) and treat co-clustered, trend-correlated features as candidate transformation products. The rationale is that genuine transformation products co-occur and co-vary with the parent compound over the time course, so unsupervised density-based clustering recovers them without needing prior structural identification.

## Related tools

- **Mass-Suite (MSS)** (Provides dm.ms_cluster (Z-score normalization + DBSCAN/OPTICS clustering) and eps_assess for knee-plot eps selection; writes cluster assignments and optional cluster reports) — https://github.com/XiminHu/mass-suite
- **Python** (Execution environment for the mss package and its Pandas/Numpy/Scikit-learn dependencies) — https://github.com/XiminHu/mass-suite
- **scikit-learn** (Backend implementation of the DBSCAN clustering algorithm invoked through dm.ms_cluster) — https://scikit-learn.org/stable/modules/generated/sklearn.cluster.DBSCAN.html

## Examples

```
from mss.dm import ms_cluster; clusters = ms_cluster(feature_table, clustering_algorithm='DBSCAN', eps=0.3, min_samples=5, dimension_reduction=True, report=True)
```

## Evaluation signals

- Clustering should prioritize ~18 potential transformation-product features and match 82% of the 11 major biotransformation products reported in the original dienogest/drospirenone study
- Z-score normalization should be confirmed as applied (dm.ms_cluster default) before clustering
- eps should be justified by the eps_assess knee plot and min_samples scaled to the feature number (example: eps=0.3, min_samples=5)
- cluster_assignments.csv is produced and contains one cluster label per input feature, with DBSCAN noise points (label -1) excluded from the prioritized set
- Optional cluster report/PCA/T-SNE output should show coherent abundance-trend groups consistent with the time-course (0, 4, 10, 29 h)

## Limitations

- DBSCAN requires manual tuning of eps and min_samples; wrong choices misclassify real products as outliers or merge unrelated trends
- Feature grouping to merge isotopes, adducts, and in-source fragments is not yet implemented, so co-clustered features may include duplicate signals of the same chemical
- Authors caution users must understand the algorithms' limitations and assumptions to avoid misinterpretation, but no specific limits are enumerated
- No version number, DOI, or archived release is cited for MSS, affecting reproducibility of the exact clustering behavior

## Evidence

- [methods] Clustering analysis is performed with the MSS function dm.ms_cluster, which uses Z-score data normalization prior to clustering by default: "Clustering analysis is performed with the MSS function dm.ms_cluster, which uses Z-score data normalization prior to clustering by default"
- [methods] eps/min_samples parameters for clustering were determined via feature numbers (min_samples) and knee plot (eps; MSS provides a function eps_assess: "determined via feature numbers (min_samples) and knee plot (eps; MSS provides a function eps_assess for this process)"
- [other] Load the exported Agilent .csv feature table from the dienogest/drospirenone biotransformation time-course (samples at 0, 4, 10, and 29 h): [evidence span withheld — anchor status: not_found]
- [other] the clustering should prioritize 18 potential transformation-product features, matching 82% of the 11 major biotransformation products reported in the original study: [evidence span withheld — anchor status: not_found]
- [readme] ms_cluster function will grouping the features into clusters using selected algorithm. Dimension reduction and cluster report are optional within the function settings.: "ms_cluster function will grouping the features into clusters using selected algorithm. Dimension reduction and cluster report are optional within the function settings."
