---
name: pytorch-tensor-shape-manipulation-for-batch-1d-ms-inputs
description: 'Use when reconstructing the MSMCE representation module: a batch of
  1D MS vectors (e.g. spectrum_dim=12000, 9900 or 15000) must be converted into a
  multi-channel embedding tensor of shape B×257×1024 for downstream classifiers.'
when_to_use_negative:
- Input is already a multi-channel feature table or an image/2D representation rather
  than raw 1D MS vectors — the reshape/concatenate pipeline assumes single-channel
  B×D spectra
- The downstream task needs interpretability of which m/z regions drive classification
  — the paper explicitly states the biological interpretation of learned embedding
  features was not investigated
- The dataset is severely imbalanced or too small — the paper warns deep models may
  fail to learn embedding representations in such extreme cases
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3862
tools:
- name: PyTorch
  role: Implement the two-layer FC encoder (LayerNorm/ReLU/Dropout), the two 1D convolutional
    layers with BatchNorm/ReLU/Dropout, the B×1×d reshape, and the channel-dimension
    concatenation as a single nn.Module trained end-to-end
- name: torchinfo
  role: Run summary() on the wrapped module to record output tensor shape and total
    learnable parameters, confirming the B×257×1024 configuration
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:000bf80ca5cd89b62b7da55f0abf3a0b02a41ae343ecb209c1004f0b0eb39a41
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - torchinfo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3862
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pytorch-tensor-shape-manipulation-for-batch-1d-ms-inputs

## Summary

Implements the MSMCE module's tensor-shape pipeline in PyTorch: a TIC-normalized, m/z-binned (0.1 Da) 1D MS vector batch of shape B×D is encoded to a B×1×d global embedding, expanded to a B×C×d multi-channel embedding via 1D convolutions (C=256, d=1024), and concatenated to the fused representation O∈R^(B×257×1024).

## When to use

Use when reconstructing the MSMCE representation module: a batch of 1D MS vectors (e.g. spectrum_dim=12000, 9900 or 15000) must be converted into a multi-channel embedding tensor of shape B×257×1024 for downstream classifiers.

## When NOT to use

- Input is already a multi-channel feature table or an image/2D representation rather than raw 1D MS vectors — the reshape/concatenate pipeline assumes single-channel B×D spectra
- The downstream task needs interpretability of which m/z regions drive classification — the paper explicitly states the biological interpretation of learned embedding features was not investigated
- The dataset is severely imbalanced or too small — the paper warns deep models may fail to learn embedding representations in such extreme cases

## Inputs

- Batch of 1D MS vectors X of shape B×D (e.g. spectrum_dim=12000 for NSCLC/CRLM, 9900 for RCC, 15000 for Canine Sarcoma), TIC-normalized and binned at 0.1 Da
- Hyperparameters: embedding_channels=256, embedding_dim=1024, in_channels=1

## Outputs

- Fused multi-channel embedding tensor O of shape B×(1+C)×d, i.e. B×257×1024
- torchinfo summary and learnable parameter count for the nn.Module

## How to apply

Load a batch of TIC-normalized, 0.1 Da-binned MS vectors X of shape B×D and pass it through a two-layer fully connected encoder (D→2048, LayerNorm, ReLU, Dropout, then 2048→d=1024) to get the global embedding E. Reshape E to E' of shape B×1×d to add a channel dimension. Apply a 1D convolution with kernel shape (C2,1,3), BatchNorm, ReLU, Dropout, then a second 1D convolution with kernel shape (C,C2,3) and padding 1 (preserving d=1024), yielding the multi-channel embedding of shape B×C×d with C=256. Concatenate E' with the conv output along the channel dimension to produce O of shape B×(1+C)×d = B×257×1024. Wrap encoder, channel embedding, and concatenation in a single nn.Module so gradients flow jointly to downstream classifiers (ResNet50/DenseNet121/EfficientNetB0/LSTM/Transformer) in end-to-end training; verify with torchinfo summary and a sample forward pass.

## Related tools

- **PyTorch** (Implement the two-layer FC encoder (LayerNorm/ReLU/Dropout), the two 1D convolutional layers with BatchNorm/ReLU/Dropout, the B×1×d reshape, and the channel-dimension concatenation as a single nn.Module trained end-to-end)
- **torchinfo** (Run summary() on the wrapped module to record output tensor shape and total learnable parameters, confirming the B×257×1024 configuration)

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 64 --epochs 64 --use_normalization
```

## Evaluation signals

- Forward pass on a sample batch runs without error
- Output tensor O has exact shape B×257×1024 (1 + embedding_channels=256 channels × embedding_dim=1024), per Eq. 7 of the Methods
- Length dimension d=1024 is preserved through both 1D convolutions (padding=1 on the second conv with kernel_size=3)
- Gradients flow jointly from downstream classifier (e.g. ResNet50) back through the module in end-to-end training
- Module summary (torchinfo) reports the expected learnable parameter count for D→2048→1024 encoder plus (C2,1,3) and (C,C2,3) convolutions

## Limitations

- Embedding dimensions and parameter optimization require dataset-specific tuning; no adaptive feature selection mechanism is provided
- Direct biological/clinical interpretability of learned embedding features (e.g. which m/z regions contribute to classification) is not investigated
- Performance may be limited for severely imbalanced class distributions or very small sample sizes; effectiveness in transfer learning scenarios is untested

## Evidence

- [methods] concatenates E' with C2 along the channel dimension to yield O∈R^(B×(1+C)×d): [evidence span withheld — anchor status: not_found]
- [abstract] we implement a feature fusion mechanism by concatenating the initial encoded representation with the multi-channel embeddings along the channel dimension: "Additionally, we implement a feature fusion mechanism by concatenating the initial encoded representation with the multi-channel embeddings along the channel dimension"
- [readme] The required data files should be put into `datasets/` folder. Note that the input of each dataset is tic normalized: "The required data files should be put into `datasets/` folder. Note that the input of each dataset is tic normalized in this implementation."
- [results] estimated to use the summary function from the Python library torchinfo.: "estimated to use the summary function from the Python library torchinfo."
- [readme] --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 64 --epochs 64 --use_normalization: "--embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 64 --epochs 64 --use_normalization"
