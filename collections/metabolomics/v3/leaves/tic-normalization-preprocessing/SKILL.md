---
name: tic-normalization-preprocessing
description: Use when raw MS spectra must be intensity-scaled per sample (TIC) before
  binning into a fixed spectrum_dim (e.g. 9900 for RCC positive mode) and fed to CNN/LSTM
  classifiers or ML baselines.
when_to_use_negative:
- The input matrix is already TIC-normalized (per the README, 'the input of each dataset
  is tic normalized in this implementation'); re-applying TIC would rescale identical
  rows redundantly.
- The analysis targets absolute analyte quantification or inter-sample total-signal
  comparison, where removing the per-sample intensity scale is undesirable.
- Class distributions are severely imbalanced or sample size is too small — the article
  notes the module's performance 'may still be limited in extreme cases, such as severely
  imbalanced class distributions.'
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3473
tools:
- name: exp_multi_channel_embedding.py (MSMCE repo)
  role: Entry-point script whose --use_normalization flag enables TIC normalization
    of input data before binning and model training
  repo: https://github.com/WoodFY/MSMCE
- name: scikit-learn
  role: Used for stratified 90/10 splitting, 6-fold CV fold splits, ML baselines (SVM,
    logistic regression, Random Forest, XGBoost), and row-normalization utilities
    (pinned scikit_learn == 1.3.0)
- name: PyTorch
  role: Implements the training pipeline that consumes the TIC-normalized, binned
    tensors (pinned pytorch == 2.3.1)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:7a20ff1ebeaf2a706326e2b7ec384dc775949c1f7f3153aa076c05f62d0e1be7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - PyTorch
  - exp_multi_channel_embedding.py (MSMCE repo)
  - pandas
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# tic-normalization-preprocessing

## Summary

Applies total ion current (TIC) normalization to rows of the processed mass spectrometry feature matrix before m/z binning (0.1 Da) and RT aggregation (10 s) so that spectra are on a comparable intensity scale prior to MSMCE or baseline classification.

## When to use

Use when raw MS spectra must be intensity-scaled per sample (TIC) before binning into a fixed spectrum_dim (e.g. 9900 for RCC positive mode) and fed to CNN/LSTM classifiers or ML baselines.

## When NOT to use

- The input matrix is already TIC-normalized (per the README, 'the input of each dataset is tic normalized in this implementation'); re-applying TIC would rescale identical rows redundantly.
- The analysis targets absolute analyte quantification or inter-sample total-signal comparison, where removing the per-sample intensity scale is undesirable.
- Class distributions are severely imbalanced or sample size is too small — the article notes the module's performance 'may still be limited in extreme cases, such as severely imbalanced class distributions.'

## Inputs

- Processed raw MS feature matrix placed in datasets/ (e.g. rcc_posion positive ion mode), pre-TIC-normalization
- Raw MS file list for stratified 90%/10% train/hold-out splitting
- CLI configuration: --use_normalization, --spectrum_dim (9900/12000/15000), --bin_size 0.1, --rt_binning_window 10

## Outputs

- TIC-normalized, 0.1 Da-binned spectrum matrix of fixed spectrum_dim ready for classifier input
- Per-model Accuracy, Precision, Recall, F1-Score rows written to results_table.csv (values in [0,1])

## How to apply

Load the processed feature matrix from the datasets/ folder and apply TIC normalization to the feature rows prior to model input, as the README notes the input of each dataset is TIC normalized in this implementation. In the MSMCE pipeline this is toggled with the --use_normalization flag of exp_multi_channel_embedding.py, and the normalized rows are then binned with m/z bin_size 0.1 Da and a 10 s retention-time binning window to a fixed spectrum dimension (9900 for rcc_posion, 12000 for NSCLC/CRLM, 15000 for canine_sarcoma_posion). Split the raw MS file list 90%/10% with stratified sampling BEFORE binning so the hold-out test set is processed identically, keeping the same normalization scale-free transform per row. The TIC-normalized, binned matrix then feeds both the MSMCE-integrated deep models (e.g. MultiChannelEmbeddingResNet50, embedding_channels 256, embedding_dim 1024) and single-channel baselines (ResNet50/DenseNet121) plus SVM, logistic regression, Random Forest and XGBoost under identical 6-fold stratified CV with a fixed random seed (default 3407). Rationale: normalization removes total-signal differences between acquisitions so the representation module learns structural rather than global-intensity features.

## Related tools

- **exp_multi_channel_embedding.py (MSMCE repo)** (Entry-point script whose --use_normalization flag enables TIC normalization of input data before binning and model training) — https://github.com/WoodFY/MSMCE
- **scikit-learn** (Used for stratified 90/10 splitting, 6-fold CV fold splits, ML baselines (SVM, logistic regression, Random Forest, XGBoost), and row-normalization utilities (pinned scikit_learn == 1.3.0))
- **PyTorch** (Implements the training pipeline that consumes the TIC-normalized, binned tensors (pinned pytorch == 2.3.1))

## Examples

```
python exp_multi_channel_embedding.py --dataset rcc_posion --spectrum_dim 9900 --k_folds 6 --embedding_channels 256 --embedding_dim 1024 --use_normalization
```

## Evaluation signals

- Row-wise invariant: after TIC normalization, each spectrum row's intensity values sum to a constant total ion current.
- The resulting spectrum dimension matches the dataset configuration: 9900 for RCC positive ion mode, 12000 for NSCLC/CRLM, 15000 for Canine Sarcoma.
- results_table.csv exists and contains Accuracy/F1 values strictly within [0,1] for each model row.
- On RCC, the pipeline should reproduce the paper's improvement pattern: DenseNet121 F1-Score rising from 0.6946 (baseline) to 0.7952 (MSMCE-integrated), p = 0.03, N = 6.
- Hold-out and CV folds use identical fixed random seed (default 3407) so normalized inputs are split identically across MSMCE and baseline runs.

## Limitations

- TIC normalization is specific to 'this implementation' (README); externally sourced raw MS data must be verified as processed/normalized before use with these scripts.
- The article notes dataset-specific tuning is required: 'the choice of embedding dimensions and parameter optimization still require dataset-specific tuning', so the normalized input dimension must be re-checked per dataset.
- Deep models may fail to learn embeddings when sample size is too small or classes are severely imbalanced, even after normalization.

## Evidence

- [readme] Note that the input of each dataset is tic normalized in this implementation.: "Note that the input of each dataset is tic normalized in this implementation."
- [readme] use_normalization flag description: "use_normalization  | Flag to enable normalization of the input data."
- [methods] TIC normalization in the RCC task workflow: [evidence span withheld — anchor status: not_found]
