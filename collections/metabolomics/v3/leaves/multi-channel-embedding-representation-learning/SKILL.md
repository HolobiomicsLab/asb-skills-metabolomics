---
name: multi-channel-embedding-representation-learning
description: Use when classifying raw, TIC-normalized 1D mass spectrometry spectra
  (e.g., NSCLC, CRLM, RCC, or Canine Sarcoma feature matrices) and single-channel
  CNN/sequence baselines underperform;
when_to_use_negative:
- The input is already a curated feature table or a non-MS data modality — MSMCE is
  designed for raw/TIC-normalized 1D MS vectors, and embedding dimensions require
  dataset-specific tuning for new domains
- The dataset is severely class-imbalanced or has very few samples — the paper notes
  performance may be limited in such extreme cases and deep models may fail to learn
  embeddings
- The goal is transfer learning across datasets/tasks — the article states transferability
  of the module has not yet been evaluated
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MSMCE (WoodFY/MSMCE)
  role: Multi-channel embedding representation module and training script exp_multi_channel_embedding.py;
    source code repository
  repo: https://github.com/WoodFY/MSMCE
- name: PyTorch
  role: Deep learning framework for end-to-end training of MSMCE-integrated classifiers
    (pytorch == 2.3.1)
- name: scikit-learn
  role: Traditional ML benchmarks (SVM, logistic regression, Random Forest) and stratified
    k-fold splitting (scikit_learn == 1.3.0)
- name: thop
  role: FLOPs calculation for comparing computational efficiency of baseline vs MSMCE-enhanced
    models via the profile function
- name: torchinfo
  role: Model size/parameter estimation via the summary function for baseline and
    MSMCE-enhanced models
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:adc2fa3572671fffd95fe57548c8e789c00579b53a0e9f32a33af54397f545ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - Adam optimizer
  - MSMCE (WoodFY/MSMCE repository)
  - MSMCE (WoodFY/MSMCE)
  - PyTorch
  - ReduceLROnPlateau
  - pandas
  - scikit-learn
  - thop
  - torchinfo
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 24
---

# multi-channel-embedding-representation-learning

## Summary

Applies the MSMCE (Multi-Channel Embedding Representation Module) to one-dimensional raw mass spectrometry vectors, learning inter-channel dependencies to produce multi-channel embedded representations that are fused with an initial encoded representation and fed to classifiers such as ResNet50, DenseNet121, EfficientNetB0, LSTM, or Transformer.

## When to use

Use when classifying raw, TIC-normalized 1D mass spectrometry spectra (e.g., NSCLC, CRLM, RCC, or Canine Sarcoma feature matrices) and single-channel CNN/sequence baselines underperform; the goal is a learnable, end-to-end multi-channel representation module instead of hand-crafted feature selection.

## When NOT to use

- The input is already a curated feature table or a non-MS data modality — MSMCE is designed for raw/TIC-normalized 1D MS vectors, and embedding dimensions require dataset-specific tuning for new domains
- The dataset is severely class-imbalanced or has very few samples — the paper notes performance may be limited in such extreme cases and deep models may fail to learn embeddings
- The goal is transfer learning across datasets/tasks — the article states transferability of the module has not yet been evaluated

## Inputs

- TIC-normalized raw MS feature matrices (m/z binned at 0.1 Da, RT-aggregated over 10 s windows) placed in datasets/ (e.g., NSCLC/CRLM spectrum_dim=12000, RCC=9900, Canine Sarcoma=15000)
- Dataset name and class count (nsclc/crlm/rcc_posion with 2 classes; canine_sarcoma_posion with 12 classes)
- Pretrained PyTorch backbone choice: ResNet50, DenseNet121, EfficientNetB0, LSTM, or Transformer

## Outputs

- Trained MSMCE-integrated model checkpoints (saved to --save_dir)
- Per-model Accuracy and macro-averaged F1-Score on the 10% hold-out test set, comparable to the paper's Tables 2-5 (e.g., MSMCE-ResNet50 accuracy 0.9785 vs baseline 0.9661 on NSCLC)
- Wilcoxon signed-rank p-values comparing MSMCE-integrated vs baseline models (e.g., p=0.03, N=6)

## How to apply

Load the TIC-normalized dataset (e.g., NSCLC/CRLM binned to spectrum_dim 12000 m/z features at 0.1 Da bins with 10 s RT window aggregation, RCC at 9900, Canine Sarcoma at 15000) into datasets/, then train the MSMCE+classifier end-to-end with exp_multi_channel_embedding.py using embedding_channels=256, embedding_dim=1024, k_folds=6, Adam (lr=1e-3, weight_decay=1e-5), class-weighted cross-entropy, batch_size 64 (32 for Canine Sarcoma), max 64 epochs, ReduceLROnPlateau (decay 0.1 after 5 stagnant epochs), and early stopping with patience 10. The module concatenates the initial encoded representation with the multi-channel embeddings along the channel dimension for feature fusion. Train the single-channel baseline (e.g., ResNet50 with in_channels=1) on identical fold splits (random_seed 3407) and compare on a fixed stratified 90/10 train/hold-out split, computing Accuracy and macro-averaged F1 and testing significance with the paired two-sided Wilcoxon signed-rank test (N=6, p<0.05).

## Related tools

- **MSMCE (WoodFY/MSMCE)** (Multi-channel embedding representation module and training script exp_multi_channel_embedding.py; source code repository) — https://github.com/WoodFY/MSMCE
- **PyTorch** (Deep learning framework for end-to-end training of MSMCE-integrated classifiers (pytorch == 2.3.1))
- **scikit-learn** (Traditional ML benchmarks (SVM, logistic regression, Random Forest) and stratified k-fold splitting (scikit_learn == 1.3.0))
- **thop** (FLOPs calculation for comparing computational efficiency of baseline vs MSMCE-enhanced models via the profile function)
- **torchinfo** (Model size/parameter estimation via the summary function for baseline and MSMCE-enhanced models)

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 64 --epochs 64 --use_normalization
```

## Evaluation signals

- NSCLC: MSMCE-ResNet50 accuracy approximates 0.9785 vs baseline ResNet50 0.9661 (+1.28%, paired Wilcoxon p=0.03, N=6); Random Forest F1 around 0.9935
- CRLM: MSMCE-DenseNet121 accuracy improves by 6.66% over DenseNet121 (p=0.03, N=6)
- RCC (positive ion mode): DenseNet121 F1-Score rises from 0.6946 (baseline) to 0.7952 (+14.48%, p=0.03, N=6)
- Canine Sarcoma (12-class): MSMCE-ResNet50 accuracy reaches approximately 0.9043 vs baseline 0.7235 (+24.99%, p=0.03, N=6)
- Paired two-sided Wilcoxon signed-rank tests (N=6 folds) yield p<0.05 for MSMCE-integrated vs baseline models, and all reported accuracy/F1 values fall in [0,1]

## Limitations

- The choice of embedding dimensions and parameter optimization requires dataset-specific tuning; no adaptive feature selection mechanism yet
- Performance may be limited in extreme cases such as severely imbalanced class distributions, and deep models may fail to learn embedding representations when sample size is too small
- The biological/clinical interpretability of learned embedding features (which m/z regions contribute to classification) has not been investigated
- Effectiveness in transfer learning scenarios and transferability across datasets/tasks not yet evaluated

## Evidence

- [abstract] MSMCE module constructs inter-channel dependencies for multi-channel MS representations: "we propose a Multi-Channel Embedding Representation Module (MSMCE), which focuses on modeling inter-channel dependencies to generate multi-channel representations of raw MS data"
- [abstract] Feature fusion by channel-wise concatenation: "we implement a feature fusion mechanism by concatenating the initial encoded representation with the multi-channel embeddings along the channel dimension"
- [results] Wilcoxon significance testing protocol: "we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines"
- [results] NSCLC headline result used for validation: "MSMCE-ResNet50 achieved an accuracy of 0.9785, a statistically significant increase of 1.28% (p = 0.03, N = 6) compared to the original model accuracy of 0.9661"
- [readme] README command line for training MSMCE-ResNet50 on NSCLC: "python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256"
- [readme] Default random seed and early stopping parameters documented in README: "random_seed        | Random seed for reproducibility (default: `3407`)."
