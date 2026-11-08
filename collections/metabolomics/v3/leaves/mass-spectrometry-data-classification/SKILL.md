---
name: mass-spectrometry-data-classification
description: Use when raw, TIC-normalized mass spectrometry feature vectors (e.g.,
  spectrum_dim 9900–15000 from m/z binning at 0.1 Da and 10 s RT windows) must be
  classified, and multi-channel representation learning should be compared against
  single-channel baselines.
when_to_use_negative:
- The dataset has severely imbalanced class distributions or too few samples — the
  article notes MSMCE performance 'may still be limited in extreme cases, such as
  severely imbalanced class distributions' and that deep models may fail to learn
  embeddings when sample size is too small
- The goal is biological/clinical interpretability of learned features (e.g., which
  m/z regions drive classification or correlation with biomarkers) — the study explicitly
  does not investigate this
- A transfer-learning or cross-dataset generalization scenario — transferability of
  the module across datasets/tasks has not yet been evaluated
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Implements the MSMCE module, ResNet50/DenseNet121/EfficientNetB0/LSTM/Transformer
    classifiers, Adam optimizer, ReduceLROnPlateau scheduler, and early-stopping loop
- name: scikit-learn
  role: Provides stratified k-fold splitting, traditional ML benchmarks (SVM, logistic
    regression, Random Forest, XGBoost wrappers), and metric computation
- name: exp_multi_channel_embedding.py
  role: Reference training/evaluation entry point of the MSMCE repository; takes --model_name,
    --dataset, --embedding_channels, --embedding_dim, --k_folds, --batch_size, --epochs
    flags
  repo: https://github.com/WoodFY/MSMCE
- name: thop
  role: FLOPs were calculated using the profile function from the Python library thop
    for baseline and MSMCE-enhanced models
- name: torchinfo
  role: Parameter counts estimated with the summary function from the Python library
    torchinfo
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:b7464dfb327a863d718022ca70a8ee13f3427bbcbe042c7c7b6fe75e523ecd98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - Adam optimizer
  - PyTorch
  - ReduceLROnPlateau
  - exp_multi_channel_embedding.py
  - pandas
  - scikit-learn
  - thop
  - torchinfo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.88
n_claims: 25
---

# mass-spectrometry-data-classification

## Summary

End-to-end classification of raw 1D mass spectrometry vectors by inserting the MSMCE multi-channel embedding module (256 channels, 1024 embedding dim) before CNN/sequence classifiers (ResNet50, DenseNet121, EfficientNetB0, LSTM, Transformer), benchmarked against single-channel baselines and traditional ML (SVM, logistic regression, Random Forest, XGBoost) with paired Wilcoxon validation.

## When to use

Use when raw, TIC-normalized mass spectrometry feature vectors (e.g., spectrum_dim 9900–15000 from m/z binning at 0.1 Da and 10 s RT windows) must be classified, and multi-channel representation learning should be compared against single-channel baselines.

## When NOT to use

- The dataset has severely imbalanced class distributions or too few samples — the article notes MSMCE performance 'may still be limited in extreme cases, such as severely imbalanced class distributions' and that deep models may fail to learn embeddings when sample size is too small
- The goal is biological/clinical interpretability of learned features (e.g., which m/z regions drive classification or correlation with biomarkers) — the study explicitly does not investigate this
- A transfer-learning or cross-dataset generalization scenario — transferability of the module across datasets/tasks has not yet been evaluated

## Inputs

- TIC-normalized MS feature matrix (spectrum_dim 12000 for NSCLC/CRLM, 9900 for RCC positive-ion mode, 15000 for Canine Sarcoma) placed in the datasets/ folder
- Raw MS file list with m/z and retention-time information for 0.1 Da binning and 10 s RT-window aggregation
- Class labels (binary for NSCLC/CRLM/RCC; 12 classes for Canine Sarcoma)
- Processed data bundle from the paper's Google Drive mirror (optional)

## Outputs

- results_table.csv with per-model Accuracy/Precision/Recall/F1-Score values in [0,1] for each model row
- Wilcoxon signed-rank test p-values (two-sided, N=6) for MSMCE vs baseline comparisons
- Bootstrap 95% confidence intervals for mean Accuracy, Precision, Recall, F1-Score from k-fold cross-validation
- Estimated peak GPU memory footprint (MB) and FLOPs for baseline and MSMCE-enhanced models

## How to apply

Preprocess raw MS files by m/z binning at 0.1 Da and aggregating over 10-second retention-time windows, then TIC-normalize the feature rows. Split the file list by stratified sampling into 90% training and 10% fixed hold-out test sets, and run 6-fold stratified cross-validation on the training set with identical fold splits across models (fixed random seed, default 3407). Train MSMCE-integrated models (embedding_channels=256, embedding_dim=1024) with the Adam optimizer (lr 1e-3, weight decay 1e-5), class-weighted cross-entropy, ReduceLROnPlateau (decay 0.1 after 5 stagnant epochs), and early stopping (patience 10) for a maximum of 64 epochs; train the single-channel baselines (in_channels=1) and traditional ML benchmarks (SVM, logistic regression, Random Forest, XGBoost with default parameters) under the same protocol. Concatenate the initial encoded representation with the multi-channel embeddings along the channel dimension as the feature-fusion step before the classifier head. Evaluate all models on the hold-out test set with Accuracy, Precision, Recall, and macro-averaged F1, then apply a two-sided paired Wilcoxon signed-rank test (N=6 folds, p<0.05 significance) comparing MSMCE-integrated models against their baselines. Write per-model metrics to results_table.csv, and optionally compute FLOPs (thop) and parameter counts (torchinfo) plus bootstrap 95% CIs from the k-fold results.

## Related tools

- **PyTorch** (Implements the MSMCE module, ResNet50/DenseNet121/EfficientNetB0/LSTM/Transformer classifiers, Adam optimizer, ReduceLROnPlateau scheduler, and early-stopping loop)
- **scikit-learn** (Provides stratified k-fold splitting, traditional ML benchmarks (SVM, logistic regression, Random Forest, XGBoost wrappers), and metric computation)
- **exp_multi_channel_embedding.py** (Reference training/evaluation entry point of the MSMCE repository; takes --model_name, --dataset, --embedding_channels, --embedding_dim, --k_folds, --batch_size, --epochs flags) — https://github.com/WoodFY/MSMCE
- **thop** (FLOPs were calculated using the profile function from the Python library thop for baseline and MSMCE-enhanced models)
- **torchinfo** (Parameter counts estimated with the summary function from the Python library torchinfo)

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset rcc_posion --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 9900 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 64 --epochs 64 --use_normalization
```

## Evaluation signals

- results_table.csv exists and contains accuracy/F1 values within [0,1] for every model row
- Paired Wilcoxon signed-rank tests yield p < 0.05 (N=6 folds) for each MSMCE-integrated model vs its baseline
- Reproduction check: DenseNet121 F1-Score on RCC rises from 0.6946 (baseline) to 0.7952 (MSMCE-integrated), p = 0.03
- Reproduction check: MSMCE-ResNet50 accuracy on NSCLC reaches ~0.9785 vs 0.9661 baseline (+1.28%); Canine Sarcoma 12-class MSMCE-ResNet50 accuracy ~0.9043 vs 0.7235 (+24.99%)
- All models (deep and traditional ML) are trained and evaluated on identical stratified fold splits with the same fixed random seed and the same 10% hold-out test set

## Limitations

- Embedding dimensions and parameter optimization require dataset-specific tuning; no adaptive feature selection mechanism is available
- Lack of biological/clinical interpretability of learned embedding features is a stated limitation
- Performance may degrade for severely imbalanced class distributions or very small sample sizes
- Reported gains can be modest for near-saturated tasks (e.g., +1.28% on NSCLC) — the RF benchmark even reached the highest F1 of 0.9935 on NSCLC

## Evidence

- [results] MSMCE-ResNet50 achieved an accuracy of 0.9785, a statistically significant increase of 1.28% (p = 0.03, N = 6) compared to the original model accuracy of 0.9661: "MSMCE-ResNet50 achieved an accuracy of 0.9785, a statistically significant increase of 1.28% (p = 0.03, N = 6) compared to the original model accuracy of 0.9661"
- [results] Wilcoxon test protocol: "we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines"
- [abstract] Feature fusion mechanism: "we implement a feature fusion mechanism by concatenating the initial encoded representation with the multi-channel embeddings along the channel dimension"
- [readme] TIC normalization note: "Note that the input of each dataset is tic normalized in this implementation."
- [readme] RCC invocation: "python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset rcc_posion --num_classes 2 --k_folds 6 --in_channels 1"
- [results] RCC F1 improvement: "on the RCC dataset, the F1-Score for the DenseNet121 increased from 0.6946 to 0.7952"
- [readme] Parameter defaults: "embedding_channels | Number of embedding channels (default: `256`)"
