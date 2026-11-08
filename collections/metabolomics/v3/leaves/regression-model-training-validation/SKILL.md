---
name: regression-model-training-validation
description: Use when a clustered HRMS feature table from a dilution/mixing experiment
  (e.g., roadway runoff diluted into other water samples) must be turned into a predictive
  regression model for source tracking and apportionment of unknown mixtures.
when_to_use_negative:
- The data are raw mzML scans — feature extraction and clustering (mssmain.get_scans,
  mssmain.peak_pick, dm.ms_cluster) must precede modeling
- The goal is peak quality assessment during import, where MSS instead integrates
  a random forest model as an optional assessing tool rather than a source-apportionment
  regressor
- There is no dilution/mixing training series — the README positions dm modeling specifically
  'for the feature clustering using dilution datasets, aiming at dilution prediction,
  source tracking and matrix effect assessment'
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3315
tools:
- name: Mass-Suite (MSS)
  role: dm.feature_model trains, validates (R² report, predicted-vs-actual plot),
    and deploys regression models on clustered HRMS features; dm.ms_cluster supplies
    the fingerprint clusters used as modeling targets
  repo: https://github.com/XiminHu/mass-suite
- name: Python
  role: Runtime language for the MSS package and its pandas/scikit-learn-based regression
    modeling
  repo: https://github.com/XiminHu/mass-suite
- name: pymzML
  role: Parses raw .mzML into Python metadata upstream of feature extraction and modeling
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:7943bcdd86405ec7dca15cfa99bbdba0a48dd65a81b1a186a4faf98c7f86d7f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - Python
  - pymzML
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# regression-model-training-validation

## Summary

Trains and validates regression models on HRMS feature clusters (e.g., dilution series of roadway runoff source fingerprints) using MSS dm.feature_model, then deploys the fitted model to predict source concentration/dilution in unknown mixed samples.

## When to use

Use when a clustered HRMS feature table from a dilution/mixing experiment (e.g., roadway runoff diluted into other water samples) must be turned into a predictive regression model for source tracking and apportionment of unknown mixtures.

## When NOT to use

- The data are raw mzML scans — feature extraction and clustering (mssmain.get_scans, mssmain.peak_pick, dm.ms_cluster) must precede modeling
- The goal is peak quality assessment during import, where MSS instead integrates a random forest model as an optional assessing tool rather than a source-apportionment regressor
- There is no dilution/mixing training series — the README positions dm modeling specifically 'for the feature clustering using dilution datasets, aiming at dilution prediction, source tracking and matrix effect assessment'

## Inputs

- Transposed pandas feature dataframe (samples as rows, HRMS features as columns) built from the pre-processed roadway runoff dilution/mixing dataset (Peter et al.) in the package 'example_data' folder
- Cluster labels from dm.ms_cluster (e.g., DBSCAN with Z-score normalization, eps=0.6, min_samples=10)
- Feature-model settings: cluster_algorithm flag, model_method (linear regression, random forest regression, or ensemble bagging regression), report flag

## Outputs

- Trained regression model for a selected fingerprint cluster
- Model assessment report with coefficient of determination and predicted-vs-actual plot
- Per-mixture predicted fold change / source concentration estimates for unknown mixed samples (e.g., source_apportionment_predictions.csv)

## How to apply

Transpose the processed HRMS feature dataframe so samples are rows and features are columns, then run MSS dm.ms_cluster (DBSCAN with Z-score normalization; e.g., eps=0.6, min_samples=10) and select a fingerprint cluster (e.g., label 0, N=587 features). Call MSS dm.feature_model with cluster_algorithm=True and model_method set to one of the supported regressors (linear regression, random forest regression, or ensemble bagging regression, e.g., 'ensemble_bagging_svc'-style settings), splitting the data into training and held-out test sets. Enable report=True to generate a model assessment report including the coefficient of determination and a predicted-vs-actual visualization. If the R² and visual agreement are acceptable, deploy the trained model via the dm.feature_model prediction workflow on unknown mixed samples to estimate fold change (predicted vs. actual concentration) per mixture, writing results (e.g., source_apportionment_predictions.csv) and report plots.

## Related tools

- **Mass-Suite (MSS)** (dm.feature_model trains, validates (R² report, predicted-vs-actual plot), and deploys regression models on clustered HRMS features; dm.ms_cluster supplies the fingerprint clusters used as modeling targets) — https://github.com/XiminHu/mass-suite
- **Python** (Runtime language for the MSS package and its pandas/scikit-learn-based regression modeling) — https://github.com/XiminHu/mass-suite
- **pymzML** (Parses raw .mzML into Python metadata upstream of feature extraction and modeling) — https://github.com/pymzml/pymzML

## Examples

```
from mss.dm import feature_model; feature_model(d_transpose, cluster_algorithm=True, model_method='ensemble_bagging_svc', report=True)
```

## Evaluation signals

- Model assessment report yields a high coefficient of determination (R²) on the held-out test set, with predicted values visually tracking actual values in the predicted-vs-actual plot
- Predicted fold change per mixture in source_apportionment_predictions.csv is plausible relative to known dilution/mixing ratios in the roadway runoff example
- The chosen cluster is a coherent fingerprint cluster (e.g., DBSCAN label 0 with N=587 features) rather than noise points
- Model_method used is one of the supported regressors: linear regression, random forest regression, or ensemble bagging regression
- README states all training data are reproducible: the example_data folder can be 'repeatly trained by users according to the settings'

## Limitations

- Chemical fingerprint-based source apportionment tool optimization is still in development, not released (listed as future work in the discussion)
- Feature grouping (merging isotopes, adducts, in-source fragments) is not yet implemented, so coeluting/adduct features may confound the regression features
- Authors caution users must understand algorithm limitations and assumptions to avoid misinterpretation, but no specific algorithm limitations are enumerated
- No Python version or dependency versions are specified in requirements ('Other requirements: none'), so model reproducibility across environments is not guaranteed

## Evidence

- [readme] After clustering, modeling function is provided to predict the dilution rate from samples and catch the behavior of individual clusters: "After clustering, modeling function is provided to predict the dilution rate from samples and catch the behavior of individual clusters"
- [readme] Currently supporting models: linear regression, random forest regression, ensemble bagging regression: "Currently supporting models: linear regression, random forest regression, ensemble bagging regression"
- [methods] Clustering analysis is performed with the MSS function dm.ms_cluster, which uses Z-score data normalization prior to clustering by default: "Clustering analysis is performed with the MSS function dm.ms_cluster, which uses Z-score data normalization prior to clustering by default"
- [methods] two complex roadway runoff samples were diluted and mixed with other water samples to mimic downstream mixing behaviors of multiple potential contaminant sources: "two complex roadway runoff samples were diluted and mixed with other water samples to mimic downstream mixing behaviors of multiple potential contaminant sources"
- [intro] MSS also provides advanced data analysis (e.g., unsupervised clustering analysis, source tracking modeling), heuristic data exploration, data mining, and predictive modeling capabilities: "MSS also provides advanced data analysis (e.g., unsupervised clustering analysis, source tracking modeling), heuristic data exploration, data mining, and predictive modeling capabilities"
