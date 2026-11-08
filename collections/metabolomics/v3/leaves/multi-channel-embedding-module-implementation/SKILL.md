---
name: multi-channel-embedding-module-implementation
description: Use when raw, TIC-normalized, m/z-binned (0.1 Da) 1D MS vectors must
  be transformed into multi-channel embedded representations to feed CNN/LSTM/Transformer
  classifiers instead of single-channel inputs.
when_to_use_negative:
- Input is already a multi-channel or image-structured representation (the module
  targets raw single-channel 1D MS vectors, in_channels=1)
- The task requires biological/clinical interpretation of which m/z regions contribute
  to classification — the article states the biological interpretation of learned
  features was not investigated
- Sample size is too small or classes are severely imbalanced — the article notes
  deep models may fail to learn embedding representations in these extreme cases
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3520
tools:
- name: PyTorch
  role: Framework for implementing the two-layer FC encoder, 1D convolutional channel
    embedding, and channel concatenation as a single nn.Module trained end-to-end
  repo: https://github.com/WoodFY/MSMCE
- name: torchinfo
  role: Used with the summary function to estimate model parameters and confirm the
    module configuration matches the described architecture
- name: thop
  role: Used with the profile function to calculate FLOPs of MSMCE-enhanced versus
    baseline models
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:771d4fc30fdfb1c291e45f07b4fa38b2db8058a3f092e0e760fda75dcb69f850
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - PyTorch
  - thop
  - torchinfo
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multi-channel-embedding-module-implementation

## Summary

Implements the MSMCE (Multi-Channel Embedding) representation module for raw 1D mass spectrometry vectors in PyTorch: a two-layer fully connected encoder (D→2048→d=1024 with LayerNorm/ReLU/Dropout) followed by two 1D convolutional channel-embedding layers (out_channels=256, kernel_size=3, padding 1), whose outputs are concatenated along the channel dimension to form a fused B×257×1024 tensor for downstream classifiers.

## When to use

Use when raw, TIC-normalized, m/z-binned (0.1 Da) 1D MS vectors must be transformed into multi-channel embedded representations to feed CNN/LSTM/Transformer classifiers instead of single-channel inputs.

## When NOT to use

- Input is already a multi-channel or image-structured representation (the module targets raw single-channel 1D MS vectors, in_channels=1)
- The task requires biological/clinical interpretation of which m/z regions contribute to classification — the article states the biological interpretation of learned features was not investigated
- Sample size is too small or classes are severely imbalanced — the article notes deep models may fail to learn embedding representations in these extreme cases

## Inputs

- Batch of 1D MS vectors X of shape B×D from TIC-normalized, m/z-binned (0.1 Da) preprocessed dataset files (e.g. NSCLC spectrum_dim=12000, RCC 9900, Canine Sarcoma 15000)
- Hyperparameters: embedding_channels=256, embedding_dim=1024, in_channels=1

## Outputs

- Fused multi-channel embedding tensor O of shape B×(1+C)×d, i.e. B×257×1024
- nn.Module (arch_msmce_module) integrable with ResNet50/DenseNet121/EfficientNetB0/LSTM/Transformer classifiers for joint end-to-end optimization

## How to apply

Wrap encoder, channel embedding, and concatenation in a single nn.Module so gradients flow jointly during end-to-end training with downstream classifiers (ResNet50/DenseNet121/EfficientNetB0/LSTM/Transformer). First pass the B×D input batch (e.g. spectrum_dim=12000, 0.1 Da bins) through linear layers mapping D→2048 then 2048→d=1024, each followed by LayerNorm, ReLU, and Dropout. Reshape the global embedding E to B×1×d, then apply a 1D conv with kernel (C2,1,3) plus BatchNorm/ReLU/Dropout, then a second conv with kernel (C,C2,3) and padding 1 so the d=1024 length is preserved, producing B×C×d with C=256 channels. Concatenate E' and the multi-channel embedding C2 along the channel dimension to yield O∈R^(B×257×1024), per the Methods (Eq. 7). Verify by running a forward pass and recording the output shape and total learnable parameters via torchinfo's summary function.

## Related tools

- **PyTorch** (Framework for implementing the two-layer FC encoder, 1D convolutional channel embedding, and channel concatenation as a single nn.Module trained end-to-end) — https://github.com/WoodFY/MSMCE
- **torchinfo** (Used with the summary function to estimate model parameters and confirm the module configuration matches the described architecture)
- **thop** (Used with the profile function to calculate FLOPs of MSMCE-enhanced versus baseline models)

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 64 --epochs 64 --use_normalization
```

## Evaluation signals

- Forward pass on a sample batch must run without error and output tensor O must have exact shape B×257×1024 as specified in the Methods (Eq. 7)
- torchinfo summary confirms learnable parameter count matches the described configuration (FC layers D→2048→1024 plus two 1D convs with out_channels 256)
- The d=1024 length dimension is preserved through the convolutional stack (padding=1 on the second conv)
- Gradients flow through encoder, channel embedding, and classifier jointly during end-to-end training with the downstream classifier
- FLOPs measured with thop's profile function show the expected computational profile for MSMCE-enhanced versus baseline models

## Limitations

- Embedding dimensions and parameter optimization require dataset-specific tuning; no adaptive feature selection mechanism yet
- Lack of biological/clinical interpretability of learned embedding features (e.g., which m/z regions contribute to classification) was not investigated
- Effectiveness in transfer learning scenarios and transferability across datasets/tasks not yet evaluated

## Evidence

- [abstract] we implement a feature fusion mechanism by concatenating the initial encoded representation with the multi-channel embeddings along the channel dimension: "we implement a feature fusion mechanism by concatenating the initial encoded representation with the multi-channel embeddings along the channel dimension"
- [methods] concatenate E' with C2 along the channel dimension to yield O∈R^(B×(1+C)×d): [evidence span withheld — anchor status: not_found]
- [results] estimated to use the summary function from the Python library torchinfo: "estimated to use the summary function from the Python library torchinfo."
- [discussion] the choice of embedding dimensions and parameter optimization still require dataset-specific tuning: "the choice of embedding dimensions and parameter optimization still require dataset-specific tuning."
- [readme] The input of each dataset is tic normalized in this implementation: "Note that the input of each dataset is tic normalized in this implementation."
