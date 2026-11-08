---
name: deep-learning-retention-time-prediction
description: Use when small-molecule ECFP fingerprints plus experimental reverse-phase
  RT values are available and the goal is to predict LC–MS retention times, compare
  ML vs. similarity-based baselines at Tanimoto thresholds, or rank candidate metabolite
  identities by predicted RT.
when_to_use_negative:
- The RT data come from a different chromatographic method than reverse-phase LC,
  unless you specifically intend to test projecting predicted RT onto other chromatographic
  methods
- No experimental RT labels exist for training — the model requires a large-scale
  dataset of experimentally acquired chromatographic RT values
- The task is MS/MS spectra prediction rather than retention time prediction; the
  paper notes the historical lack of datasets for in silico spectra prediction, which
  this RT dataset does not address
tools:
- name: Keras
  role: Framework used to build and train the 4-layer fully connected regression model
    (1000/500/200/100 units, relu, L2 0.0001, adam lr 0.01, MSE loss, 20 epochs, batch
    size 35)
- name: k-nearest neighbors regression
  role: 'Naive baseline: retrieves the k=3 most similar training molecules by Tanimoto
    similarity and uses the median of their experimental RT as the prediction'
- name: Tanimoto similarity
  role: Similarity metric used to stratify validation molecules by threshold (95/90/80/70/50%)
    and to select the 3 nearest training-set counterparts for the k-NN baseline
- name: Wilcoxon rank test
  role: Paired statistical test comparing naive k-NN vs. DLM relative RT errors within
    each Tanimoto-threshold group
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:59c9c894d998ffc53ce4db001dc82ec983e38bf4bc1fd1be215c3f60ad0d1318
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Keras
  - Tanimoto similarity
  - Wilcoxon rank test
  - k-nearest neighbors regression
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# deep-learning-retention-time-prediction

## Summary

Trains a Keras feed-forward regression model on the METLIN SMRT dataset (80,038 small molecules with experimentally acquired reverse-phase LC retention times) using ECFP fingerprints as input, to predict retention time (RT) in seconds and benchmark it against a naive k-NN median-RT baseline.

## When to use

Use when small-molecule ECFP fingerprints plus experimental reverse-phase RT values are available and the goal is to predict LC–MS retention times, compare ML vs. similarity-based baselines at Tanimoto thresholds, or rank candidate metabolite identities by predicted RT.

## When NOT to use

- The RT data come from a different chromatographic method than reverse-phase LC, unless you specifically intend to test projecting predicted RT onto other chromatographic methods
- No experimental RT labels exist for training — the model requires a large-scale dataset of experimentally acquired chromatographic RT values
- The task is MS/MS spectra prediction rather than retention time prediction; the paper notes the historical lack of datasets for in silico spectra prediction, which this RT dataset does not address

## Inputs

- METLIN SMRT dataset of small molecules with experimentally acquired reverse-phase chromatography retention times (RT in seconds)
- ECFP fingerprints for each molecule (feature representation)
- Tanimoto similarity thresholds (e.g., 95, 90, 80, 70, 50%) for stratifying validation molecules against training counterparts

## Outputs

- Per-molecule predicted RT values (seconds) for the validation set from the Keras model
- Per-molecule relative RT errors for both the DLM and the k-NN (k=3) median-RT baseline, grouped by Tanimoto similarity threshold
- Group summary statistics (median, IQR) and paired Wilcoxon rank test P values per threshold, e.g., written to knn_dlm_error_comparison.tsv
- Ranked candidate identity lists based on predicted retention time

## How to apply

Load the METLIN SMRT dataset (RT in seconds plus ECFP fingerprints) and randomly split molecules 75% training / 25% validation. Train a Keras deep-learning regression model with 4 fully connected hidden layers of 1000, 500, 200, and 100 units (relu activation, L2 regularization 0.0001, adam optimizer with learning rate 0.01, MSE loss, 20 epochs, batch size 35) and predict RT for the validation set. As a baseline, for each validation molecule retrieve the k=3 most similar training molecules by Tanimoto similarity and take the median of their experimental RT; stratify both methods' relative RT errors by similarity thresholds (95, 90, 80, 70, 50%) and compare them with paired Wilcoxon rank tests. The rationale is that predicted RT can then be used to rank and filter putative candidates from accurate mass search in metabolomics experiments, where in the paper 70% of cases had the correct identity ranked among the top 3 candidates.

## Related tools

- **Keras** (Framework used to build and train the 4-layer fully connected regression model (1000/500/200/100 units, relu, L2 0.0001, adam lr 0.01, MSE loss, 20 epochs, batch size 35))
- **k-nearest neighbors regression** (Naive baseline: retrieves the k=3 most similar training molecules by Tanimoto similarity and uses the median of their experimental RT as the prediction)
- **Tanimoto similarity** (Similarity metric used to stratify validation molecules by threshold (95/90/80/70/50%) and to select the 3 nearest training-set counterparts for the k-NN baseline)
- **Wilcoxon rank test** (Paired statistical test comparing naive k-NN vs. DLM relative RT errors within each Tanimoto-threshold group)

## Examples

```
from keras.models import Sequential; from keras.layers import Dense; model = Sequential([Dense(1000, activation='relu', kernel_regularizer='l2(1e-4)'), Dense(500, activation='relu'), Dense(200, activation='relu'), Dense(100, activation='relu'), Dense(1)]); model.compile(optimizer=keras.optimizers.Adam(lr=0.01), loss='mse'); model.fit(X_train_ecfp, y_train_rt_s, epochs=20, batch_size=35)
```

## Evaluation signals

- Recovered significance pattern matches the reported one: paired Wilcoxon rank test P < 0.0001 at the 90, 80, 70, and 50% Tanimoto thresholds, and no statistically significant mean-error difference at the 95% threshold
- Split is a random 75% training / 25% validation partition of the 80,038-molecule METLIN SMRT dataset, and RT units are seconds
- The k-NN baseline uses exactly k=3 most similar training molecules with median experimental RT, not the mean
- In candidate-ranking mode, approximately 70% of cases should rank the correct molecular identity among the top 3 candidates
- Output file (e.g., knn_dlm_error_comparison.tsv) contains per-molecule errors, group summaries (median, IQR), and Wilcoxon P values

## Limitations

- At very high similarity (95% Tanimoto threshold) the DLM does not significantly outperform the naive k-NN median-RT baseline, so the learned model adds no value for near-identical analogs
- The model was trained on reverse-phase chromatography RT; generalization to other chromatographic methods requires explicit evaluation of projecting predicted RT onto other CMs
- Historically, the lack of publicly available experimental RT datasets has hampered such ML approaches; results depend on the scale (80,038 molecules) and provenance of the METLIN SMRT dataset

## Evidence

- [abstract] abstract: "Here we introduce the METLIN small molecule retention time (SMRT) dataset, an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules."
- [intro] intro: "in 70% of the cases, the correct molecular identity was ranked among the top 3 candidates based on their predicted retention time"
- [intro] intro-2: "Further, we focused on the ability to rank and filter putative candidates in real metabolomics experiments based on accurate mass search and predicted RT"
- [intro] intro-3: "We analyzed the model’s predictive ability along with the ability to project predicted RT onto other CMs"
