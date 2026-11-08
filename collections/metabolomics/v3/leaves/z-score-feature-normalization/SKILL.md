---
name: z-score-feature-normalization
description: Use when a multi-sample HRMS feature table (e.g., Agilent .csv export
  with 136 features from the dienogest/drospirenone time-course at 0, 4, 10, 29 h)
  is about to be clustered by dm.ms_cluster and per-feature abundance trends must
  be made comparable before DBSCAN eps/min_samples are set.
when_to_use_negative:
- The input is still raw .mzML data — feature extraction (mssmain.get_scans, mssmain.peak_pick
  with peak width, replicated peak, and baseline-noise filters) must occur first,
  before normalization/clustering.
- The analysis goal preserves absolute abundance or fold-change magnitudes (e.g.,
  dilution prediction or background subtraction on peak area fold-change criteria),
  since Z-scoring removes the intensity scale and keeps only relative trend shape.
- There is only one sample per feature — the per-feature standard deviation σ is undefined,
  so z=(x-μ)/σ cannot be computed.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Mass-Suite (MSS)
  role: Provides dm.ms_cluster, which applies Z-score data normalization prior to
    clustering by default, and eps_assess for setting the DBSCAN eps on the normalized
    data
  repo: https://github.com/XiminHu/mass-suite
- name: Python
  role: Runtime environment in which the MSS package and its clustering/normalization
    functions are executed (pip install mass-suite; import mss)
  repo: https://github.com/XiminHu/mass-suite
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:7216ab492e999a1317cd8ae245ec3e1b384142586a361a3d2c953bafd87b69f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - Python
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Z-score feature normalization

## Summary

Standardizes per-feature abundances across samples with z=(x-μ)/σ, the default preprocessing inside the MSS dm.ms_cluster function, so that unsupervised clustering (DBSCAN/OPTICS) groups features by abundance-trend shape rather than absolute intensity scale, eliminating skewness and kurtosis in the feature table.

## When to use

Use when a multi-sample HRMS feature table (e.g., Agilent .csv export with 136 features from the dienogest/drospirenone time-course at 0, 4, 10, 29 h) is about to be clustered by dm.ms_cluster and per-feature abundance trends must be made comparable before DBSCAN eps/min_samples are set.

## When NOT to use

- The input is still raw .mzML data — feature extraction (mssmain.get_scans, mssmain.peak_pick with peak width, replicated peak, and baseline-noise filters) must occur first, before normalization/clustering.
- The analysis goal preserves absolute abundance or fold-change magnitudes (e.g., dilution prediction or background subtraction on peak area fold-change criteria), since Z-scoring removes the intensity scale and keeps only relative trend shape.
- There is only one sample per feature — the per-feature standard deviation σ is undefined, so z=(x-μ)/σ cannot be computed.

## Inputs

- Pre-processed feature table exported from Agilent HRMS data as .csv (features x samples, e.g., 136 features after blank/control subtraction from the dienogest/drospirenone biotransformation time-course at 0, 4, 10, and 29 h)
- Feature abundance matrix (formats supported by Pandas, e.g., .csv/.xlsx) suitable for mss.dm.ms_cluster

## Outputs

- Z-score-normalized feature matrix consumed internally by dm.ms_cluster prior to DBSCAN clustering
- Per-feature cluster assignments written to cluster_assignments.csv (including outlier label -1 for features excluded as noise)

## How to apply

Take the pre-processed feature table (after blank subtraction, replicate filtering, and control subtraction reduced it to 136 features) and, for each feature row, compute z=(x-μ)/σ across the time-course samples. In MSS this is done automatically: dm.ms_cluster applies Z-score data normalization prior to clustering by default, so no separate call is needed, but the analyst must confirm the normalization is active before tuning DBSCAN parameters (eps determined via the MSS eps_assess knee plot; min_samples set from the feature number, e.g., eps=0.3, min_samples=5). The rationale is that normalization removes intensity-scale differences between features so clustering reflects the shape of the abundance trend across samples, and outliers (cluster label -1) can be excluded. If desired, downstream PCA or T-SNE dimension reduction and a cluster report visualize the normalized structure before writing cluster_assignments.csv.

## Related tools

- **Mass-Suite (MSS)** (Provides dm.ms_cluster, which applies Z-score data normalization prior to clustering by default, and eps_assess for setting the DBSCAN eps on the normalized data) — https://github.com/XiminHu/mass-suite
- **Python** (Runtime environment in which the MSS package and its clustering/normalization functions are executed (pip install mass-suite; import mss)) — https://github.com/XiminHu/mass-suite

## Examples

```
from mss.dm import ms_cluster; ms_cluster(feature_table, cluster_algorithm='DBSCAN', eps=0.3, min_samples=5, report=True)
```

## Evaluation signals

- After normalization, each feature row has mean ≈ 0 and standard deviation ≈ 1 across the time-course samples (0, 4, 10, 29 h).
- On the example dataset, the subsequent DBSCAN clustering (eps=0.3, min_samples=5) prioritizes 18 potential transformation-product features.
- The clustering result matches 82% of the 11 major biotransformation products reported in the original dienogest/drospirenone study.
- Features assigned the DBSCAN outlier label (-1) are excluded from cluster outputs rather than forced into clusters.
- The optional cluster report/PCA/T-SNE visualization shows grouping structure on the normalized abundances, not on raw peak areas.

## Limitations

- Z-score normalization is applied silently as the dm.ms_cluster default — users who need unnormalized abundance trends must override this, and the authors caution users must develop fundamental understanding about the algorithms, their limitations and assumptions to avoid misinterpretation.
- Normalization is designed for trend-shape comparison; it is not a feature-grouping step (merging isotopes, adducts, and in-source fragments is not yet implemented in MSS and is listed as future work).
- The downstream clustering quality depends on correctly choosing eps (via the eps_assess knee plot) and min_samples from the feature number — e.g., the example dataset's 136 features — and parameters would typically differ for each dataset.

## Evidence

- [methods] Clustering analysis is performed with the MSS function dm.ms_cluster, which uses Z-score data normalization prior to clustering by default: "Clustering analysis is performed with the MSS function dm.ms_cluster, which uses Z-score data normalization prior to clustering by default"
- [methods] determined via feature numbers (min_samples) and knee plot (eps; MSS provides a function eps_assess for this process).: "determined via feature numbers (min_samples) and knee plot (eps; MSS provides a function eps_assess for this process)."
- [readme] ms_cluster function will grouping the features into clusters using selected algorithm. Dimension reduction and cluster report are optional within the function settings.: "ms_cluster function will grouping the features into clusters using selected algorithm. Dimension reduction and cluster report are optional within the function settings."
- [readme] This is a data analysis module using cluster algorithm to find grouping features, detecting noises and mining for dilution prediction/source tracking.: "This is a data analysis module using cluster algorithm to find grouping features, detecting noises and mining for dilution prediction/source tracking."
