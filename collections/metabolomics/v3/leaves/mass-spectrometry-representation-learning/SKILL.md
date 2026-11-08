---
name: mass-spectrometry-representation-learning
description: Use when raw MS vectors (e.g. TIC-normalized spectra with spectrum_dim
  9900–15000) need a richer feature representation than a single-channel 1D vector,
  to improve classification accuracy and computational efficiency of CNN classifiers
  like ResNet50 or DenseNet121.
when_to_use_negative:
- When the dataset is severely class-imbalanced or sample size is too small — the
  article warns deep models may fail to learn embedding representations in such extreme
  cases.
- When aggressive feature selection into a low-dimensional feature table is already
  required or adequate; MSMCE is designed to learn representations directly from raw
  MS data.
- When biological/clinical interpretability of which m/z regions drive classification
  is the primary goal — the article notes the direct biological interpretation of
  learned features has not been deeply investigated.
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: thop
  role: Calculates FLOPs of MSMCE-integrated versus baseline models using the profile
    function to demonstrate computational efficiency.
- name: torchinfo
  role: Estimates model parameters and GPU memory footprint via the summary function
    for MSMCE-enhanced and baseline architectures.
- name: WoodFY/MSMCE
  role: Reference implementation repository containing exp_multi_channel_embedding.py
    for training MSMCE-integrated models on NSCLC, CRLM, RCC, and Canine Sarcoma datasets.
  repo: https://github.com/WoodFY/MSMCE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:414666fe1a94cb6f14071cdac7a414ba8d4ff31c3e73d5948c8fa32b4fcb5ec6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - WoodFY/MSMCE
  - thop
  - torchinfo
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mass-spectrometry-representation-learning

## Summary

A skill for representing raw, TIC-normalized 1D mass spectrometry vectors as learnable multi-channel embeddings via the MSMCE module, which models inter-channel dependencies and concatenates the encoded representation with multi-channel embeddings to boost downstream classification accuracy and training stability.

## When to use

Use when raw MS vectors (e.g. TIC-normalized spectra with spectrum_dim 9900–15000) need a richer feature representation than a single-channel 1D vector, to improve classification accuracy and computational efficiency of CNN classifiers like ResNet50 or DenseNet121.

## When NOT to use

- When the dataset is severely class-imbalanced or sample size is too small — the article warns deep models may fail to learn embedding representations in such extreme cases.
- When aggressive feature selection into a low-dimensional feature table is already required or adequate; MSMCE is designed to learn representations directly from raw MS data.
- When biological/clinical interpretability of which m/z regions drive classification is the primary goal — the article notes the direct biological interpretation of learned features has not been deeply investigated.

## Inputs

- TIC-normalized raw mass spectrometry spectrum vectors (spectrum_dim e.g. 9900, 12000, or 15000)
- Labeled dataset in positive ion mode (e.g. NSCLC, CRLM, RCC, Canine Sarcoma 12-class)
- Optional command-line hyperparameters: embedding_channels (256), embedding_dim (1024), in_channels (1), batch_size, epochs (64)

## Outputs

- Learnable multi-channel embedding representations of raw MS data (fused tensor from channel-wise concatenation of E' and C2)
- Classification metrics per model (Accuracy, Precision, Recall, F1-Score with macro unweighted averaging) with 6-fold cross-validation results and bootstrap 95% confidence intervals
- Estimated peak GPU memory footprint (MB) and FLOPs per model variant
- Trained end-to-end MSMCE-integrated classifier checkpoints (e.g. MultiChannelEmbeddingResNet50)

## How to apply

Load TIC-normalized spectrum vectors (e.g. spectrum_dim 15000 for Canine Sarcoma, 9900 for RCC positive-ion mode) and pass them through the MSMCE module: a two-layer fully connected Encoder (LayerNorm, ReLU, Dropout) producing an initial encoded representation, followed by two stacked 1D convolution layers (kernel_size 3, batch normalization, ReLU, Dropout) that model inter-channel dependencies to generate multi-channel embeddings (configured at 256 embedding channels and embedding dimension 1024). Fuse features by concatenating the initial encoded representation E' with the multi-channel embedding C2 along the channel dimension, then feed the fused tensor to a classifier such as ResNet50, DenseNet121, EfficientNetB0, LSTM, or Transformer in an end-to-end framework. Train for up to 64 epochs with Adam (lr 1e-3, weight decay 1e-5), class-weighted cross-entropy, ReduceLROnPlateau scheduler (patience 5, factor 0.1), and early stopping (patience 10). Run 6-fold cross-validation (N = 6) and assess significance against baselines using the paired two-sided Wilcoxon signed-rank test with p < 0.05; quantify efficiency with thop (FLOPs) and torchinfo (summary).

## Related tools

- **thop** (Calculates FLOPs of MSMCE-integrated versus baseline models using the profile function to demonstrate computational efficiency.)
- **torchinfo** (Estimates model parameters and GPU memory footprint via the summary function for MSMCE-enhanced and baseline architectures.)
- **WoodFY/MSMCE** (Reference implementation repository containing exp_multi_channel_embedding.py for training MSMCE-integrated models on NSCLC, CRLM, RCC, and Canine Sarcoma datasets.) — https://github.com/WoodFY/MSMCE

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset canine_sarcoma_posion --num_classes 12 --k_folds 6 --in_channels 1 --spectrum_dim 15000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 32 --epochs 64 --use_normalization
```

## Evaluation signals

- On the NSCLC dataset MSMCE-ResNet50 should reach Accuracy ~0.9785 (vs baseline 0.9661, +1.28%, p = 0.03, N = 6) — a statistically significant improvement by paired two-sided Wilcoxon signed-rank test at p < 0.05.
- Ablation: adding Channel Embedding alone should raise Canine Sarcoma accuracy to ~0.8834 (F1 0.8888), and full MSMCE must exceed that (paper reports 0.9043 for MSMCE-ResNet50).
- FLOPs and parameter counts measured with thop and torchinfo should show reduced compute and GPU memory relative to baseline CNNs.
- Training curves should be stable with no 'mode collapse' typically seen for sequence models; early stopping (patience 10) should not trigger prematurely.
- Bootstrap 95% confidence intervals from 6-fold cross-validation on the hold-out test set should be reported for Accuracy, Precision, Recall, and F1-Score for all evaluated models.

## Limitations

- The choice of embedding dimensions and parameter optimization still require dataset-specific tuning; no adaptive feature selection mechanism yet.
- Performance may be limited in extreme cases such as severely imbalanced class distributions, and deep models may fail to learn embedding representations when sample size is too small.
- The direct biological interpretation of the learned embedding features (e.g. which m/z regions contribute to classification) has not been deeply investigated.
- Effectiveness in transfer learning scenarios and transferability across datasets/tasks has not yet been evaluated.

## Evidence

- [abstract] we propose a Multi-Channel Embedding Representation Module (MSMCE), which focuses on modeling inter-channel dependencies to generate multi-channel representations of raw MS data: "we propose a Multi-Channel Embedding Representation Module (MSMCE), which focuses on modeling inter-channel dependencies to generate multi-channel representations of raw MS data"
- [abstract] we implement a feature fusion mechanism by concatenating the initial encoded representation with the multi-channel embeddings along the channel dimension: "we implement a feature fusion mechanism by concatenating the initial encoded representation with the multi-channel embeddings along the channel dimension"
- [results] Experiments were conducted on the Canine Sarcoma dataset: "experiments were conducted on the Canine Sarcoma dataset"
- [results] FLOPs were calculated using the profile function from the Python library thop.: "FLOPs were calculated using the profile function from the Python library thop."
- [results] estimated to use the summary function from the Python library torchinfo: "estimated to use the summary function from the Python library torchinfo"
- [results] MSMCE-ResNet50 achieved an accuracy of 0.9785, a statistically significant increase of 1.28% (p = 0.03, N = 6): "MSMCE-ResNet50 achieved an accuracy of 0.9785, a statistically significant increase of 1.28% (p = 0.03, N = 6)"
- [readme] The input of each dataset is tic normalized in this implementation.: "Note that the input of each dataset is tic normalized in this implementation."
- [readme] MSMCE command for Canine Sarcoma: "python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset canine_sarcoma_posion --num_classes 12 --k_folds 6 --in_channels 1 --spectrum_dim 15000 --embedding_channels"
