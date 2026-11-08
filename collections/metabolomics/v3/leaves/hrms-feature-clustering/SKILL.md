---
name: hrms-feature-clustering
description: Use when a pre-processed HRMS feature table (e.g., Agilent .csv export,
  features x samples) needs grouping by abundance-trend similarity — e.g., to flag
  candidate transformation products across a time course or isolate contaminant source
  fingerprints from dilution/mixing series.
when_to_use_negative:
- Input is still raw .mzML data — run mssmain.get_scans and mssmain.peak_pick (with
  peak width, replicated-peak, and background-subtraction filters) to build the feature
  table first
- The goal is feature annotation or MS/MS identification (use mss.frag with MoNA MassBank
  searching), not grouping by abundance trend
- Feature labels are already known and a supervised approach is intended — use dm.feature_model
  regression instead of unsupervised clustering
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Mass-Suite (MSS)
  role: Provides dm.ms_cluster (Z-score normalization + DBSCAN/OPTICS clustering),
    eps_assess for knee-plot eps selection, and optional cluster reporting
  repo: https://github.com/XiminHu/mass-suite
- name: scikit-learn
  role: Backend implementing the DBSCAN and OPTICS clustering algorithms called by
    ms_cluster (required dependency 0.23.0+)
- name: pandas
  role: Holds the transposed feature table dataframe consumed by the dm module (required
    dependency 1.0.3+)
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:5d9176b306fd4d451487dc1bfd277b1f498b5edbfe803d0e6c3dbd77c7e6fff2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - Python
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# hrms-feature-clustering

## Summary

Unsupervised clustering of HRMS non-targeted analysis feature tables using the Mass-Suite (MSS) dm.ms_cluster function, which Z-score normalizes feature abundances and groups features with similar abundance trends via DBSCAN (or OPTICS) to support transformation-product prioritization and source-fingerprint discovery.

## When to use

Use when a pre-processed HRMS feature table (e.g., Agilent .csv export, features x samples) needs grouping by abundance-trend similarity — e.g., to flag candidate transformation products across a time course or isolate contaminant source fingerprints from dilution/mixing series.

## When NOT to use

- Input is still raw .mzML data — run mssmain.get_scans and mssmain.peak_pick (with peak width, replicated-peak, and background-subtraction filters) to build the feature table first
- The goal is feature annotation or MS/MS identification (use mss.frag with MoNA MassBank searching), not grouping by abundance trend
- Feature labels are already known and a supervised approach is intended — use dm.feature_model regression instead of unsupervised clustering

## Inputs

- Pre-processed HRMS feature table (.csv/.xlsx, Pandas-compatible), features x samples, after blank/control subtraction (e.g., Agilent .csv export with 136 features from the dienogest/drospirenone time course)
- Roadway runoff source dilution/mixing HRMS dataset (Peter et al.) from the package 'example_data' folder for fingerprint clustering

## Outputs

- Per-feature cluster assignments (cluster_assignments.csv) with DBSCAN labels, noise features labeled -1
- Optional cluster report/visualization and PCA or T-SNE dimension-reduction plots
- Fingerprint cluster label (e.g., Cluster 0, N=587 features) ready for dm.feature_model source-apportionment training

## How to apply

Load the exported feature table into a transposed pandas dataframe compatible with the MSS dm module. Z-score normalize each feature (z=(x-μ)/σ, the dm.ms_cluster default) to eliminate skewness and kurtosis in abundance distributions. Choose the DBSCAN algorithm (OPTICS and a non-parametric hypothesis test via trend_calc are alternatives) and tune parameters: run MSS's eps_assess function to obtain the knee plot for eps and set min_samples from the feature count — e.g., eps=0.3 with min_samples=5 grouped the 136-feature dienogest/drospirenone biotransformation time course (0, 4, 10, 29 h), while the roadway runoff dilution dataset used eps=0.6 with min_samples=10. Optionally enable PCA or T-SNE dimension reduction and the cluster report/visualization to inspect structure, then write per-feature cluster labels (cluster_assignments.csv). Rationale: clustering by abundance trend isolates co-behaving features — outlier/noise features get DBSCAN label -1 and are excluded, while the retained clusters serve as transformation-product candidates or source fingerprint inputs for downstream dm.feature_model prediction.

## Related tools

- **Mass-Suite (MSS)** (Provides dm.ms_cluster (Z-score normalization + DBSCAN/OPTICS clustering), eps_assess for knee-plot eps selection, and optional cluster reporting) — https://github.com/XiminHu/mass-suite
- **scikit-learn** (Backend implementing the DBSCAN and OPTICS clustering algorithms called by ms_cluster (required dependency 0.23.0+))
- **pandas** (Holds the transposed feature table dataframe consumed by the dm module (required dependency 1.0.3+))

## Examples

```
from mss import dm; clusters = dm.ms_cluster(d_transpose, cluster_algorithm='DBSCAN', eps=0.3, min_samples=5)
```

## Evaluation signals

- The biotransformation case study should prioritize 18 potential transformation-product features, matching 82% of the 11 major biotransformation products reported in the original study
- Z-score normalization (z=(x-μ)/σ) is applied by default before clustering — verify abundances are standardized, not raw
- eps was selected via the MSS eps_assess knee plot, not arbitrarily; per-example settings differ (eps=0.3/min_samples=5 for the 136-feature time course; eps=0.6/min_samples=10 for the runoff dilution series)
- Output is a per-feature cluster label file (cluster_assignments.csv); DBSCAN noise/outlier features are excluded from clusters (label -1)
- Fingerprint cluster size is reproducible, e.g., Cluster label = 0 with N = 587 features in the source-tracking example

## Limitations

- Parameter settings are dataset-specific — the paper notes parameter settings would typically be different for each example, so eps/min_samples must be re-tuned (knee plot) for new data
- Feature grouping to merge isotopes, adducts, and in-source fragments is not yet implemented in MSS and is listed as future work, so clusters may mix chemically related features
- Authors caution users must develop fundamental understanding of the algorithms, and their limitations and assumptions, to avoid misinterpretation of clustering results

## Evidence

- [methods] Clustering analysis is performed with the MSS function dm.ms_cluster, which uses Z-score data normalization prior to clustering by default: "Clustering analysis is performed with the MSS function dm.ms_cluster, which uses Z-score data normalization prior to clustering by default"
- [methods] eps via knee plot: "determined via feature numbers (min_samples) and knee plot (eps; MSS provides a function eps_assess for this process)."
- [abstract] advanced capabilities: "(e.g., unsupervised clustering analyses, a machine learning-based source tracking and apportionment tool)"
- [readme] README on ms_cluster options: "ms_cluster function will grouping the features into clusters using selected algorithm. Dimension reduction and cluster report are optional within the function settings."
