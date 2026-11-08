---
name: end-to-end-representation-module-design-for-ms-classification
description: Use when raw 1D MS spectra (TIC-normalized, m/z-binned at 0.1 Da, e.g.
  spectrum_dim 12000/9900/15000) must be transformed into multi-channel feature tensors
  feeding CNN/LSTM/Transformer classifiers, replacing single-channel or hand-crafted
  feature selection.
when_to_use_negative:
- When input is already a feature table or pre-selected m/z peaks rather than raw
  spectra — the module is designed to learn representations directly from raw MS data
  without aggressive feature selection.
- When the sample size is too small or class distributions are severely imbalanced
  — the paper notes deep models may fail to learn embedding representations in these
  extreme cases.
- When dataset-specific adaptive feature selection or interpretability of which m/z
  regions drive classification is required — embedding dimensions need dataset-specific
  tuning and biological interpretability was not established.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Framework for implementing the two-layer FC encoder, 1D convolutional channel-embedding
    layers, and concatenation as a single nn.Module with joint gradient flow
  repo: https://github.com/WoodFY/MSMCE
- name: torchinfo
  role: Summarize the implemented module (parameter count, layer shapes) via the summary
    function to confirm the output shape B×257×1024
- name: thop
  role: Profile function used to calculate FLOPs of baseline and MSMCE-enhanced models
    for computational-efficiency comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:c035d24fef8ec10287b37510c15349e2c1a13eed5d1613f2099819f937f732b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - PyTorch
  - thop
  - torchinfo
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# end-to-end-representation-module-design-for-ms-classification

## Summary

Design and implement a learnable multi-channel embedding representation module (MSMCE) that converts raw one-dimensional mass spectrometry (MS) vectors into fused multi-channel embeddings for downstream deep classifiers, trained end-to-end with joint optimization.

## When to use

Use when raw 1D MS spectra (TIC-normalized, m/z-binned at 0.1 Da, e.g. spectrum_dim 12000/9900/15000) must be transformed into multi-channel feature tensors feeding CNN/LSTM/Transformer classifiers, replacing single-channel or hand-crafted feature selection.

## When NOT to use

- When input is already a feature table or pre-selected m/z peaks rather than raw spectra — the module is designed to learn representations directly from raw MS data without aggressive feature selection.
- When the sample size is too small or class distributions are severely imbalanced — the paper notes deep models may fail to learn embedding representations in these extreme cases.
- When dataset-specific adaptive feature selection or interpretability of which m/z regions drive classification is required — embedding dimensions need dataset-specific tuning and biological interpretability was not established.

## Inputs

- Batch of raw 1D MS vectors X of shape B×D (TIC-normalized, m/z-binned at 0.1 Da; e.g. spectrum_dim=12000 for NSCLC/CRLM, 9900 for RCC, 15000 for Canine Sarcoma)
- Configuration parameters: embedding_channels=256, embedding_dim=1024, in_channels=1, num_classes, batch_size, epochs=64

## Outputs

- Fused multi-channel embedding tensor O of shape B×257×1024 (1 global + 256 convolutional channels × d=1024)
- Trained nn.Module checkpoint integrable with downstream classifiers (e.g. MultiChannelEmbeddingResNet50)
- torchinfo summary (parameter count) and thop profile (FLOPs, peak GPU memory) for the module

## How to apply

Load a batch X of 1D MS vectors (B×D) preprocessed with TIC normalization and 0.1 Da m/z binning. Pass X through a two-layer fully connected encoder (Linear D→2048 with LayerNorm, ReLU, Dropout; then Linear 2048→d with d=1024) to get global embedding E, reshaped to B×1×d. Apply a first 1D convolution (out_channels=C2, in_channels=1, kernel_size=3) with BatchNorm/ReLU/Dropout, then a second 1D convolution (out_channels=256, in_channels=C2, kernel_size=3, padding=1) preserving length d, yielding multi-channel embedding C2 of shape B×256×1024. Concatenate E' with C2 along the channel dimension to output O∈R^(B×257×1024). Wrap encoder, channel embedding, and concatenation in a single nn.Module so gradients flow jointly with downstream classifiers (ResNet50, DenseNet121, EfficientNetB0, LSTM, Transformer) during end-to-end training, using k_folds=6 cross-validation and the paired Wilcoxon signed-rank test (N=6, p<0.05) to compare against baselines.

## Related tools

- **PyTorch** (Framework for implementing the two-layer FC encoder, 1D convolutional channel-embedding layers, and concatenation as a single nn.Module with joint gradient flow) — https://github.com/WoodFY/MSMCE
- **torchinfo** (Summarize the implemented module (parameter count, layer shapes) via the summary function to confirm the output shape B×257×1024)
- **thop** (Profile function used to calculate FLOPs of baseline and MSMCE-enhanced models for computational-efficiency comparison)

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 64 --epochs 64 --use_normalization
```

## Evaluation signals

- Forward pass on a sample batch must run without error and output tensor O must have exact shape B×257×1024 as specified in Methods (Eq. 7).
- torchinfo summary confirms the layer configuration (FC encoder D→2048→1024, conv channels 1→C2→256, kernel_size 3, padding 1) and total learnable parameter count.
- Downstream MSMCE-enhanced classifier (e.g. MSMCE-ResNet50 on NSCLC) should show accuracy improvement over baseline — e.g. 0.9785 vs 0.9661, a 1.28% statistically significant increase (p = 0.03, N = 6, paired Wilcoxon signed-rank test).
- Gradients flow through the whole module end-to-end (encoder, convolutions, concatenation) when training with a downstream classifier over k_folds=6 cross-validation.
- thop/torchinfo profiled FLOPs and peak GPU memory of the MSMCE model should be recorded and compared against the baseline model.

## Limitations

- The choice of embedding dimensions and parameter optimization still require dataset-specific tuning; no adaptive feature selection mechanism exists.
- The direct biological interpretation of learned embedding features (which m/z regions contribute to classification, correlation with biomarkers) has not been investigated.
- Performance may be limited in extreme cases such as severely imbalanced class distributions; deep models may fail to learn embeddings when sample size is too small.
- Effectiveness in transfer learning scenarios and transferability across datasets/tasks has not yet been evaluated.
- Embedding dimensions and number of channels are fixed (C=256, d=1024) in the reference implementation, not learned adaptively.

## Evidence

- [methods] The MSMCE module first compresses a D-dimensional MS vector via a two-layer fully connected encoder (LayerNorm, ReLU, Dropout) to a global embedding E, reshapes it to B×1×d: [evidence span withheld — anchor status: not_found]
- [methods] Concatenate E' and C2 along the channel dimension to produce the fused output tensor O of shape B x (1+C) x d, i.e. B x 257 x 1024: [evidence span withheld — anchor status: not_found]
- [readme] we propose a Multi-Channel Embedding Representation Module (MSMCE), which focuses on modeling inter-channel dependencies to generate multi-channel representations of raw MS data: "we propose a Multi-Channel Embedding Representation Module (MSMCE), which focuses on modeling inter-channel dependencies to generate multi-channel representations of raw MS data"
- [readme] python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256 --embedding_dim 1024: "python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256"
- [results] MSMCE-ResNet50 achieved an accuracy of 0.9785, a statistically significant increase of 1.28% (p = 0.03, N = 6) compared to the original model accuracy of 0.9661: "MSMCE-ResNet50 achieved an accuracy of 0.9785, a statistically significant increase of 1.28% (p = 0.03, N = 6) compared to the original model accuracy of 0.9661"
- [discussion] the choice of embedding dimensions and parameter optimization still require dataset-specific tuning.: "the choice of embedding dimensions and parameter optimization still require dataset-specific tuning."
