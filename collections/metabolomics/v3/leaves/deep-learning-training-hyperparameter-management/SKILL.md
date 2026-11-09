---
name: deep-learning-training-hyperparameter-management
description: Use when training CNN or sequence models on TIC-normalized raw MS spectra
  (e.g. NSCLC, CRLM, RCC, Canine Sarcoma) where fold-level reproducibility and significance
  testing against a baseline are required.
when_to_use_negative:
- When the task requires a single train/test split without cross-validation or significance
  testing (e.g. quick exploratory prototyping)
- When the input data is already feature-selected tabular data rather than raw TIC-normalized
  spectra — MSMCE is designed for raw MS vector representation, not pre-extracted
  feature tables
- When sample size is too small for 6-fold CV to be meaningful, or when class distributions
  are severely imbalanced such that the paper warns performance 'may be limited'
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: thop
  role: Profile function used to compute FLOPs of baseline and MSMCE-enhanced models
    to evaluate computational efficiency alongside training.
- name: torchinfo
  role: Summary function used to estimate model size (peak GPU memory footprint) alongside
    training results.
- name: PyTorch
  role: Underlying deep learning framework (version 2.3.1) executing the training
    loop, ReduceLROnPlateau scheduler, and early stopping.
  repo: https://github.com/WoodFY/MSMCE
- name: scikit-learn
  role: Used (version 1.3.0) for baseline classifier comparisons (SVM, logistic regression,
    Random Forest, XGBoost) on the same splits.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:bc204fe6f6f701830c18e5a740ec0c4541be1b93aa49049880445da076eb5a09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - PyTorch
  - scikit-learn
  - thop
  - torchinfo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# deep-learning-training-hyperparameter-management

## Summary

Manages the end-to-end training protocol for the MSMCE deep learning classifiers on raw mass spectrometry data: 6-fold cross-validation, Adam (lr 1e-3, weight decay 1e-5), ReduceLROnPlateau (decay 0.1 after 5 stagnant epochs), early stopping with patience 10, and a 64-epoch cap — with paired Wilcoxon signed-rank significance testing across folds.

## When to use

Use when training CNN or sequence models on TIC-normalized raw MS spectra (e.g. NSCLC, CRLM, RCC, Canine Sarcoma) where fold-level reproducibility and significance testing against a baseline are required.

## When NOT to use

- When the task requires a single train/test split without cross-validation or significance testing (e.g. quick exploratory prototyping)
- When the input data is already feature-selected tabular data rather than raw TIC-normalized spectra — MSMCE is designed for raw MS vector representation, not pre-extracted feature tables
- When sample size is too small for 6-fold CV to be meaningful, or when class distributions are severely imbalanced such that the paper warns performance 'may be limited'

## Inputs

- Processed raw MS dataset under datasets/ (TIC-normalized, e.g. spectrum_dim 12000/9900/15000)
- Model architecture choice: ResNet50, DenseNet121, EfficientNetB0, LSTM, Transformer, or MultiChannelEmbedding~ variant
- Hyperparameter configuration: embedding_channels, embedding_dim, spectrum_dim, num_classes, batch_size, k_folds, epochs, device
- Random seed (default 3407) for reproducibility

## Outputs

- Trained model checkpoints saved to save_dir
- Per-fold metrics (Accuracy, Precision, Recall, F1-Score) on the hold-out test set
- Paired Wilcoxon signed-rank test p-values (N = 6, two-sided) for MSMCE vs baseline
- Bootstrap 95% confidence intervals for mean performance metrics

## How to apply

Instantiate the model (MultiChannelEmbeddingResNet50 or baseline variants like ResNet50/DenseNet121) with the MSMCE module using embedding_channels 256 and embedding_dim 1024, then train via exp_multi_channel_embedding.py with the Adam optimizer at learning rate 1e-3 and weight decay 1e-5. Split data into 6 folds (k_folds 6) for cross-validation, feeding spectrum_dim-sized TIC-normalized inputs (e.g. 12000 for NSCLC/CRLM, 9900 for RCC, 15000 for Canine Sarcoma) in batches of 64 (or 32 for the 12-class Canine Sarcoma task). Apply ReduceLROnPlateau with a decay factor of 0.1 triggered after 5 stagnant epochs, and enable early stopping with patience 10 against validation performance, capped at 64 epochs. After training, run the paired Wilcoxon signed-rank test comparing MSMCE-augmented against baseline models across the 6 folds (two-sided, p < 0.05) to quantify improvement; compute Accuracy, Precision, Recall and F1 on the hold-out test set.

## Related tools

- **thop** (Profile function used to compute FLOPs of baseline and MSMCE-enhanced models to evaluate computational efficiency alongside training.)
- **torchinfo** (Summary function used to estimate model size (peak GPU memory footprint) alongside training results.)
- **PyTorch** (Underlying deep learning framework (version 2.3.1) executing the training loop, ReduceLROnPlateau scheduler, and early stopping.) — https://github.com/WoodFY/MSMCE
- **scikit-learn** (Used (version 1.3.0) for baseline classifier comparisons (SVM, logistic regression, Random Forest, XGBoost) on the same splits.)

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset canine_sarcoma_posion --num_classes 12 --k_folds 6 --spectrum_dim 15000 --embedding_channels 256 --embedding_dim 1024 --batch_size 32 --epochs 64 --use_normalization
```

## Evaluation signals

- Reproducing the reported MSMCE-ResNet50 accuracy on Canine Sarcoma of ~0.9043 vs baseline 0.7235 (a 24.99% improvement, p = 0.03, N = 6)
- Paired Wilcoxon signed-rank test yields p < 0.05 for MSMCE-integrated model vs its baseline on the same folds
- All 6 folds converge within the 64-epoch cap without early stopping firing prematurely; ReduceLROnPlateau triggers decay after 5 stagnant epochs as expected
- Bootstrap 95% CIs on Accuracy/F1 overlap or improve upon the published supplementary values
- FLOPs and GPU memory footprint (via thop / torchinfo) match the reported efficiency improvements for MSMCE-augmented models

## Limitations

- Embedding dimensions and parameter optimization require dataset-specific tuning; no adaptive feature selection mechanism is provided.
- Performance may degrade with severely imbalanced class distributions or very small sample sizes, where deep models may fail to learn embedding representations.
- Lack of interpretability of learned embedding features — which m/z regions contribute to classification — is not investigated.
- Effectiveness in transfer learning scenarios or transferability across datasets/tasks has not been evaluated.

## Evidence

- [results] Paired Wilcoxon signed-rank test protocol: "we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines"
- [readme] Canine Sarcoma training command with explicit hyperparameters: "python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset canine_sarcoma_posion --num_classes 12 --k_folds 6 --in_channels 1 --spectrum_dim 15000 --embedding_channels"
- [supplementary] Scheduler and optimizer configuration context: "Tabulated summary of estimated peak GPU memory footprint (MB) and Floating-Point Operations (FLOPs) for baseline and MSMCE-enhanced deep learning models."
- [intro] Computational efficiency motivation tied to training: "While improving classification accuracy, the MSMCE module also contributes to computational efficiency."
- [discussion] Dataset-specific tuning caveat: "the choice of embedding dimensions and parameter optimization still require dataset-specific tuning."
- [results] Train/eval protocol for ablations: "model variants were trained on the entire training set (not subjected to k-fold splitting) and their performance was subsequently evaluated on the independent hold-out test set"
