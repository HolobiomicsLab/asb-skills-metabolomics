---
name: ml-based-source-apportionment-modeling
description: Use when downstream environmental samples contain mixtures of known contaminant
  sources (e.g., roadway runoff diluted/mixed with other waters) and you have a dilution/mixing
  HRMS training series to learn source fingerprints for apportionment.
when_to_use_negative:
- Input is raw vendor data not yet feature-extracted — run mssmain.peak_pick, feature
  filtering, and alignment first; this skill operates on transposed feature tables
- No dilution/mixing training series of known source concentrations exists — the regression
  model requires a fingerprint training scenario (the source apportionment tool optimization
  is still in development per the article)
- Goal is molecular identification rather than source apportionment — use MSS frag
  module/MoNA MassBank searching instead
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: Mass-Suite (MSS)
  role: Provides dm.ms_cluster (DBSCAN fingerprint clustering with Z-score normalization)
    and dm.feature_model (regression training, assessment report, and prediction workflow)
    used for source tracking and apportionment
  repo: https://github.com/XiminHu/mass-suite
- name: Python
  role: Runtime environment and pandas dataframes (transposed feature tables) feeding
    the MSS dm module workflows
- name: scikit-learn
  role: Backend for DBSCAN/OPTICS clustering and the regression models (linear, random
    forest, ensemble bagging) invoked through MSS dm functions
- name: pymzML
  role: Upstream parsing of raw .mzML files into Python metadata via mssmain.get_scans
    before feature extraction and modeling
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:c8ca93359bfb769ad0549e3717a835e31b96fccaa156cd0d6cdd9f037fa754fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - Python
  - pymzML
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# ml-based-source-apportionment-modeling

## Summary

Train and deploy a machine-learning regression model on HRMS chemical 'fingerprint' clusters (via Mass-Suite's dm module) to predict contaminant source presence/concentration in mixed environmental samples, enabling source tracking and apportionment.

## When to use

Use when downstream environmental samples contain mixtures of known contaminant sources (e.g., roadway runoff diluted/mixed with other waters) and you have a dilution/mixing HRMS training series to learn source fingerprints for apportionment.

## When NOT to use

- Input is raw vendor data not yet feature-extracted — run mssmain.peak_pick, feature filtering, and alignment first; this skill operates on transposed feature tables
- No dilution/mixing training series of known source concentrations exists — the regression model requires a fingerprint training scenario (the source apportionment tool optimization is still in development per the article)
- Goal is molecular identification rather than source apportionment — use MSS frag module/MoNA MassBank searching instead

## Inputs

- Pre-processed HRMS feature table from dilution/mixing training samples (e.g., roadway runoff diluted/mixed with water samples, Peter et al. dataset) as a transposed pandas dataframe
- HRMS feature table for unknown mixed downstream samples to predict on
- Cluster labels from dm.ms_cluster DBSCAN run (Z-score normalized, eps=0.6, min_samples=10)
- Selected fingerprint cluster designation (e.g., cluster label 0, N=587)

## Outputs

- Trained regression model (ensemble random forest / ensemble bagging / linear regression) on fingerprint features
- Model assessment report with R^2 (coefficient of determination) and predicted-vs-actual visualization for the held-out test set
- Per-mixture source presence/concentration (fold change) predictions for unknown samples
- source_apportionment_predictions.csv with predictions and model report plots

## How to apply

Load the pre-processed dilution/mixing HRMS dataset (e.g., the packaged Peter et al. roadway runoff dataset) into a transposed pandas dataframe and run MSS dm.ms_cluster with Z-score normalization and DBSCAN (eps=0.6, min_samples=10; use MSS eps_assess and the knee plot to tune eps). Select a fingerprint cluster (e.g., label 0, N=587) and fit a supervised regression with MSS dm.feature_model — ensemble random forest regression by default, with linear regression and ensemble bagging regression as supported alternatives (model_method='ensemble_bagging' style) — splitting into training and test sets with report=True. Judge fit by the model report's coefficient of determination and predicted-vs-actual plot on the held-out test set. Deploy the trained model via the dm.feature_model prediction workflow on unknown mixed samples to estimate source fold change/concentration, writing per-mixture predictions and report plots to source_apportionment_predictions.csv.

## Related tools

- **Mass-Suite (MSS)** (Provides dm.ms_cluster (DBSCAN fingerprint clustering with Z-score normalization) and dm.feature_model (regression training, assessment report, and prediction workflow) used for source tracking and apportionment) — https://github.com/XiminHu/mass-suite
- **Python** (Runtime environment and pandas dataframes (transposed feature tables) feeding the MSS dm module workflows)
- **scikit-learn** (Backend for DBSCAN/OPTICS clustering and the regression models (linear, random forest, ensemble bagging) invoked through MSS dm functions)
- **pymzML** (Upstream parsing of raw .mzML files into Python metadata via mssmain.get_scans before feature extraction and modeling) — https://github.com/pymzml/pymzML

## Examples

```
from mss import dm; feature_model(d_transpose, cluster_algorithm=True, model_method='ensemble_bagging_svc', report=True)
```

## Evaluation signals

- Model report on the held-out test set shows a high coefficient of determination (R^2) between predicted and actual concentrations
- Predicted-vs-actual visualization points fall near the 1:1 line for the training/test series
- DBSCAN (eps=0.6, min_samples=10) isolates a coherent fingerprint cluster (e.g., label 0, N=587) whose features track the dilution/mixing series
- source_apportionment_predictions.csv contains a fold-change (predicted vs. actual concentration) value for every mixed sample with no missing entries
- Predictions on unknown mixtures are non-degenerate (not constant) and consistent with the expected presence/concentration of each source in the mixing design

## Limitations

- Chemical fingerprint-based source apportionment tool optimization is still in development and not fully released
- Feature grouping (merging isotopes, adducts, in-source fragments) is not yet implemented, so fingerprint features may contain redundant MS features
- Users must understand algorithm limitations and assumptions (e.g., DBSCAN eps/min_samples sensitivity) to avoid misinterpretation
- Supported models are limited to linear regression, random forest regression, and ensemble bagging regression; deep-learning backends (Keras/TensorFlow/PyTorch) are optional and not part of the documented workflow

## Evidence

- [methods] Clustering analysis is performed with the MSS function dm.ms_cluster, which uses Z-score data normalization prior to clustering by default: "Clustering analysis is performed with the MSS function dm.ms_cluster, which uses Z-score data normalization prior to clustering by default"
- [methods] two complex roadway runoff samples were diluted and mixed with other water samples to mimic downstream mixing behaviors of multiple potential contaminant sources: "two complex roadway runoff samples were diluted and mixed with other water samples to mimic downstream mixing behaviors of multiple potential contaminant sources"
- [intro] MSS also provides advanced data analysis (e.g., unsupervised clustering analysis, source tracking modeling), heuristic data exploration, data mining, and predictive modeling capabilities: "MSS also provides advanced data analysis (e.g., unsupervised clustering analysis, source tracking modeling), heuristic data exploration, data mining, and predictive modeling capabilities"
- [readme] After clustering, modeling function is provided to predict the dilution rate from samples and catch the behavior of individual clusters: "After clustering, modeling function is provided to predict the dilution rate from samples and catch the behavior of individual clusters"
- [readme] Currently supporting models: linear regression, random forest regression, ensemble bagging regression: "Currently supporting models: linear regression, random forest regression, ensemble bagging regression"
- [readme] User can use the model and cluster information for new sample prediction: "User can use the model and cluster information for new sample prediction"
