---
name: eps-knee-plot-parameter-tuning
description: Use when running MSS dm.ms_cluster (DBSCAN/OPTICS) on a normalized feature
  table and the DBSCAN eps and min_samples parameters need to be chosen from the data
  instead of arbitrary defaults.
when_to_use_negative:
- The input is raw .mzML data rather than an already-extracted, pre-processed feature
  table — feature extraction (mssmain.get_scans, mssmain.peak_pick, peak quality filters)
  must run first.
- The clustering algorithm is non-parametric (e.g., MSS trend_calc hypothesis testing)
  or OPTICS was selected and does not require a fixed eps; the knee-plot eps is specifically
  for DBSCAN.
- The feature table has not been normalized — knee distances computed on unnormalized
  (skewed) abundances will not reflect the Z-score space dm.ms_cluster actually clusters
  in.
edam_operation: http://edamontology.org/operation_3432
tools:
- name: Mass-Suite (MSS)
  role: Provides eps_assess for knee-plot-based eps determination and dm.ms_cluster
    for DBSCAN/OPTICS clustering with Z-score normalization
  repo: https://github.com/XiminHu/mass-suite
- name: Python
  role: Language environment in which MSS (mss.dm module) and its scikit-learn-based
    clustering algorithms are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:30a88bbd4e854e6c9df7a8f7ab0a33ad4e423b521af31af066122fbc73128247
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - Python
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# eps-knee-plot-parameter-tuning

## Summary

A data-driven parameter selection step for MSS unsupervised clustering: the DBSCAN eps is chosen from a knee plot generated with the MSS eps_assess function, and min_samples is set from the feature count, before running dm.ms_cluster on a Z-score-normalized HRMS feature table.

## When to use

Use when running MSS dm.ms_cluster (DBSCAN/OPTICS) on a normalized feature table and the DBSCAN eps and min_samples parameters need to be chosen from the data instead of arbitrary defaults.

## When NOT to use

- The input is raw .mzML data rather than an already-extracted, pre-processed feature table — feature extraction (mssmain.get_scans, mssmain.peak_pick, peak quality filters) must run first.
- The clustering algorithm is non-parametric (e.g., MSS trend_calc hypothesis testing) or OPTICS was selected and does not require a fixed eps; the knee-plot eps is specifically for DBSCAN.
- The feature table has not been normalized — knee distances computed on unnormalized (skewed) abundances will not reflect the Z-score space dm.ms_cluster actually clusters in.

## Inputs

- Exported feature table (.csv, e.g. from Agilent export) with pre-processed features after blank and control subtraction
- Z-score-normalized feature abundance matrix (the dm.ms_cluster default normalization)
- Number of features in the table (used to set min_samples)

## Outputs

- Knee plot from MSS eps_assess from which eps is read (eps=0.3 on the example dataset)
- Chosen DBSCAN parameters (eps, min_samples) to feed into dm.ms_cluster
- Per-feature cluster assignments (e.g., cluster_assignments.csv), with low-density points excluded as outliers

## How to apply

After exporting a processed feature table (e.g., the 136-feature dienogest/drospirenone biotransformation table after blank and control subtraction), first Z-score normalize abundances, z=(x−μ)/σ, which dm.ms_cluster applies by default, to eliminate skewness and kurtosis. Then use the MSS eps_assess function to compute a knee plot of sorted neighbor distances and read the eps at the knee point (eps=0.3 on the example dataset). Set min_samples according to the number of features (min_samples=5 for the 136-feature example). Rerun eps_assess whenever the dataset, normalization, or preprocessing changes, since the knee position is dataset-specific. Pass the chosen eps and min_samples to dm.ms_cluster (optionally with PCA/T-SNE dimension reduction) to group features with similar abundance trends and label low-density outliers.

## Related tools

- **Mass-Suite (MSS)** (Provides eps_assess for knee-plot-based eps determination and dm.ms_cluster for DBSCAN/OPTICS clustering with Z-score normalization) — https://github.com/XiminHu/mass-suite
- **Python** (Language environment in which MSS (mss.dm module) and its scikit-learn-based clustering algorithms are executed)

## Examples

```
from mss.dm import ms_cluster; clusters = ms_cluster(feature_table, cluster_algorithm='DBSCAN', eps=0.3, min_samples=5, dimension_reduction=True, report=True)
```

## Evaluation signals

- On the example dataset, the tuned parameters (eps=0.3, min_samples=5) should make clustering prioritize 18 potential transformation-product features.
- Cluster assignments should match 82% of the 11 major biotransformation products reported in the original dienogest/drospirenone study.
- The knee plot should show a clear elbow in the distance curve; eps should be read at that knee, not fixed arbitrarily.
- Features with dissimilar abundance trends should be labeled as outliers (-1 in DBSCAN) rather than forced into clusters.
- Outputs should be a complete per-feature cluster assignment table (cluster_assignments.csv) covering all normalized features, with an optional cluster report/PCA/T-SNE visualization consistent with the assignments.

## Limitations

- Parameter settings are dataset-specific; the article notes that 'parameter settings would typically be different for each example' (DEMO folder), so eps=0.3 and min_samples=5 apply to the 136-feature example, not universally.
- The article's authors 'strongly encourage users to develop fundamental understanding about the algorithms, and their limitations and assumptions' to avoid misinterpreting clustering results.
- Feature grouping (merging isotopes, adducts, in-source fragments) is not yet implemented, so clusters may contain multiple features of the same chemical.
- No MSS version, Python version, or dependency versions are specified in the article/README, so exact eps_assess behavior may drift across package updates.

## Evidence

- [methods] determined via feature numbers (min_samples) and knee plot (eps; MSS provides a function eps_assess for this process).: "determined via feature numbers (min_samples) and knee plot (eps; MSS provides a function eps_assess for this process)."
- [methods] Clustering analysis is performed with the MSS function dm.ms_cluster, which uses Z-score data normalization prior to clustering by default: "Clustering analysis is performed with the MSS function dm.ms_cluster, which uses Z-score data normalization prior to clustering by default"
- [readme] This is a data analysis module using cluster algorithm to find grouping features, detecting noises and mining for dilution prediction/source tracking.: "This is a data analysis module using cluster algorithm to find grouping features, detecting noises and mining for dilution prediction/source tracking."
- [readme] ms_cluster function will grouping the features into clusters using selected algorithm. Dimension reduction and cluster report are optional within the function settings.: "ms_cluster function will grouping the features into clusters using selected algorithm. Dimension reduction and cluster report are optional within the function settings."
- [discussion] we do strongly encourage users to develop fundamental understanding about the algorithms, and their limitations and assumptions: "we do strongly encourage users to develop fundamental understanding about the algorithms, and their limitations and assumptions"
