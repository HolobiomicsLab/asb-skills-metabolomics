---
name: contaminant-source-tracking-prediction
description: Use when HRMS feature data from known contaminant sources (e.g., roadway
  runoff) mixed at known dilutions is available and the goal is to predict source
  contribution in unknown mixed environmental samples.
when_to_use_negative:
- When the task is general HRMS feature extraction or annotation from raw .mzML files
  — this skill operates downstream on a pre-processed, transposed feature dataframe
- When no dilution/mixing training series of known sources exists — the dm.feature_model
  workflow requires fingerprint training data with known source gradients
- When feature grouping (merging isotopes, adducts, in-source fragments) is required
  first — MSS does not yet implement this, and ungrouped features may confound fingerprint
  clustering
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Mass-Suite (MSS)
  role: Provides dm.ms_cluster (DBSCAN clustering with Z-score normalization), dm.feature_model
    (regression training/prediction with reports), and eps_assess for source tracking
    and apportionment
  repo: https://github.com/XiminHu/mass-suite
- name: Python
  role: Execution environment; the workflow uses pandas dataframes and scikit-learn-based
    models via the mss package
  repo: https://github.com/XiminHu/mass-suite
- name: pymzML
  role: Upstream parser used by MSS mssmain.get_scans to import .mzML raw data before
    feature extraction and fingerprint modeling
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:26cc450b39c93cea151b1a7a90609a236ec2739e4df4b7e0e42a650cc772d2fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - Python
  - pymzML
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# contaminant-source-tracking-prediction

## Summary

Trains a machine-learning model on HRMS 'fingerprint' features from dilution/mixing series (e.g., roadway runoff diluted into other waters) to predict source presence and concentration (apportionment) in unknown downstream mixtures, using the Mass-Suite (MSS) dm module.

## When to use

Use when HRMS feature data from known contaminant sources (e.g., roadway runoff) mixed at known dilutions is available and the goal is to predict source contribution in unknown mixed environmental samples.

## When NOT to use

- When the task is general HRMS feature extraction or annotation from raw .mzML files — this skill operates downstream on a pre-processed, transposed feature dataframe
- When no dilution/mixing training series of known sources exists — the dm.feature_model workflow requires fingerprint training data with known source gradients
- When feature grouping (merging isotopes, adducts, in-source fragments) is required first — MSS does not yet implement this, and ungrouped features may confound fingerprint clustering

## Inputs

- Pre-processed HRMS feature table (e.g., CSV/Excel exported from MSS feature extraction) of source dilution/mixing series, loaded transposed into a pandas dataframe
- Roadway runoff source dilution/mixing dataset (Peter et al.) from the MSS package 'example_data' folder
- Unknown mixed downstream water samples for which source apportionment is to be predicted

## Outputs

- Fingerprint cluster labels from dm.ms_cluster (e.g., Cluster label = 0, N = 587 features)
- Trained regression model (random forest / ensemble bagging / linear) with model assessment report (R², predicted vs. actual plot)
- Per-mixture source concentration predictions written to source_apportionment_predictions.csv (predicted fold change vs. actual)

## How to apply

Load the pre-processed dilution/mixing HRMS dataset (e.g., the Peter et al. roadway runoff dataset from MSS's 'example_data' folder) as a transposed pandas dataframe compatible with the MSS dm module. Run dm.ms_cluster with Z-score normalization and DBSCAN (eps = 0.6, min_samples = 10) to group features into fingerprint clusters — eps and min_samples can be guided by MSS's eps_assess knee-plot function. Select a fingerprint cluster (e.g., label 0, N = 587 features) and train a predictive model with dm.feature_model using ensemble random forest regression (alternatives: linear regression, ensemble bagging regression), splitting into training and test sets with report=True. Validate on the held-out test set via the generated performance report (coefficient of determination; predicted vs. actual plot), then deploy the trained model on unknown mixed samples to estimate fold change (predicted vs. actual concentration) per mixture, writing predictions to source_apportionment_predictions.csv.

## Related tools

- **Mass-Suite (MSS)** (Provides dm.ms_cluster (DBSCAN clustering with Z-score normalization), dm.feature_model (regression training/prediction with reports), and eps_assess for source tracking and apportionment) — https://github.com/XiminHu/mass-suite
- **Python** (Execution environment; the workflow uses pandas dataframes and scikit-learn-based models via the mss package) — https://github.com/XiminHu/mass-suite
- **pymzML** (Upstream parser used by MSS mssmain.get_scans to import .mzML raw data before feature extraction and fingerprint modeling) — https://github.com/pymzml/pymzML

## Examples

```
from mss import dm; clusters = dm.ms_cluster(d_transpose, algorithm='DBSCAN', eps=0.6, min_samples=10); model = dm.feature_model(d_transpose, cluster_algorithm=True, model_method='ensemble_bagging_regression', report=True)
```

## Evaluation signals

- Cluster selection sanity: the chosen fingerprint cluster (e.g., label 0) should contain a large coherent feature set (e.g., N = 587) whose dilution behavior tracks the source gradient
- Model report coefficient of determination (R²) on the held-out test set should be high, with predicted vs. actual points falling near the 1:1 line
- Predicted fold change per mixture (predicted vs. actual concentration) in source_apportionment_predictions.csv should be close to 1 for known mixtures
- eps and min_samples choices should be justified via the MSS eps_assess knee plot rather than arbitrary selection

## Limitations

- Chemical fingerprint-based source apportionment tool optimization is still in development, not released — the packaged example is a demonstration scenario
- Feature grouping to merge isotopes, adducts and in-source fragments is not yet implemented in MSS and is listed as future work
- Users must understand algorithm limitations and assumptions to avoid misinterpretation; the article does not enumerate specific algorithm limitations
- No archived release (e.g., Zenodo DOI) or version pinning is cited, and dependency versions are unspecified ('Other requirements: none'), which affects reproducibility

## Evidence

- [methods] two complex roadway runoff samples were diluted and mixed with other water samples to mimic downstream mixing behaviors of multiple potential contaminant sources: "two complex roadway runoff samples were diluted and mixed with other water samples to mimic downstream mixing behaviors of multiple potential contaminant sources"
- [methods] Clustering analysis is performed with the MSS function dm.ms_cluster, which uses Z-score data normalization prior to clustering by default: "Clustering analysis is performed with the MSS function dm.ms_cluster, which uses Z-score data normalization prior to clustering by default"
- [methods] determined via feature numbers (min_samples) and knee plot (eps; MSS provides a function eps_assess for this process).: "determined via feature numbers (min_samples) and knee plot (eps; MSS provides a function eps_assess for this process)."
- [intro] MSS also provides advanced data analysis (e.g., unsupervised clustering analysis, source tracking modeling), heuristic data exploration, data mining, and predictive modeling capabilities: "MSS also provides advanced data analysis (e.g., unsupervised clustering analysis, source tracking modeling), heuristic data exploration, data mining, and predictive modeling capabilities"
- [readme] feature_model(d_transpose, cluster_algorithm=True, model_method='ensemble_bagging_svc', report=True): "feature_model(d_transpose, cluster_algorithm=True, model_method='ensemble_bagging_svc', report=True)"
- [readme] The mass-suite package is also providing a data mining option for the feature clustering using dilution datasets, aiming at dilution prediction, source tracking and matrix effect assessment.: "The mass-suite package is also providing a data mining option for the feature clustering using dilution datasets, aiming at dilution prediction, source tracking and matrix effect assessment."
