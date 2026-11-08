---
name: 1d-convolutional-layer-configuration-for-ms-vectors
description: Use when a single-channel 1D MS vector (e.g. B x 1 x 1024 after FC encoding,
  TIC-normalized, m/z-binned at 0.1 Da) must be expanded into multi-channel embeddings
  for CNN/sequence classifiers.
when_to_use_negative:
- Input is already a multi-channel feature table or image-like tensor — the conv stack
  here expects a single-channel (B x 1 x d) reshaped embedding
- Very small sample sizes or severely imbalanced class distributions, where the paper
  warns deep embedding models may fail to learn representations
- Task requires biological interpretability of which m/z regions contribute to classification
  — the paper lists this as a limitation not addressed by the module
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Implement the (C2,1,3) and (C,C2,3) 1D convolutions with BatchNorm/ReLU/Dropout
    and the channel-dimension concatenation as a single nn.Module
  repo: https://github.com/WoodFY/MSMCE
- name: torchinfo
  role: Run the summary function to count learnable parameters and confirm the configured
    layer shapes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:d0eb11439e4daae835b42bd5e35e5f85b0bbca243280a1bbe3440437cd0fdb42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - PyTorch
  - torchinfo
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# 1D convolutional layer configuration for MS vectors

## Summary

Stack two 1D convolutional layers (kernel size 3, padding 1, BatchNorm/ReLU/Dropout) on reshaped TIC-normalized MS vectors to transform a single-channel global embedding into a 256-channel embedding of length 1024, enabling inter-channel dependency modeling per the MSMCE module.

## When to use

Use when a single-channel 1D MS vector (e.g. B x 1 x 1024 after FC encoding, TIC-normalized, m/z-binned at 0.1 Da) must be expanded into multi-channel embeddings for CNN/sequence classifiers.

## When NOT to use

- Input is already a multi-channel feature table or image-like tensor — the conv stack here expects a single-channel (B x 1 x d) reshaped embedding
- Very small sample sizes or severely imbalanced class distributions, where the paper warns deep embedding models may fail to learn representations
- Task requires biological interpretability of which m/z regions contribute to classification — the paper lists this as a limitation not addressed by the module

## Inputs

- Batch of 1D TIC-normalized MS vectors X of shape B x D (m/z-binned at 0.1 Da; e.g. spectrum_dim 12000 for NSCLC/CRLM, 9900 for RCC, 15000 for Canine Sarcoma)
- Global embedding E of shape B x d from the two-layer FC encoder (2048 hidden units, d=1024), reshaped to B x 1 x d

## Outputs

- Multi-channel embedding tensor of shape B x C x d (B x 256 x 1024 with embedding_channels=256)
- Fused representation O of shape B x (1+C) x d (B x 257 x 1024) after channel-dimension concatenation with the reshaped global embedding E'
- Layer/parameter summary from torchinfo (summary) for parameter counting

## How to apply

First obtain the global embedding E from the two-layer FC encoder (D -> 2048 with LayerNorm/ReLU/Dropout, then 2048 -> d=1024) and reshape it to B x 1 x d. Apply a first 1D convolution with kernel shape (C2, 1, 3) — out_channels=C2, in_channels=1, kernel_size=3 — followed by BatchNorm, ReLU, and Dropout, then a second 1D convolution with kernel shape (C, C2, 3) and padding=1 so the length dimension d=1024 is preserved, yielding C of shape B x C x d with C=256. The choice of kernel_size=3 with padding 1 keeps spatial length constant while the channel expansion from 1 to C2 to C builds the inter-channel dependencies the paper's MSMCE module relies on. Wrap the encoder, channel embedding, and channel-dimension concatenation as one nn.Module so gradients flow jointly during end-to-end training with downstream classifiers (ResNet50/DenseNet121/EfficientNetB0/LSTM/Transformer). Validate with a forward pass plus a torchinfo summary, checking the fused output is exactly B x (1+C) x d = B x 257 x 1024.

## Related tools

- **PyTorch** (Implement the (C2,1,3) and (C,C2,3) 1D convolutions with BatchNorm/ReLU/Dropout and the channel-dimension concatenation as a single nn.Module) — https://github.com/WoodFY/MSMCE
- **torchinfo** (Run the summary function to count learnable parameters and confirm the configured layer shapes)

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 64 --epochs 64 --use_normalization
```

## Evaluation signals

- Forward pass runs without error on a sample batch of TIC-normalized MS vectors
- Output tensor O has exact shape B x 257 x 1024 (B x (1+C) x d) as specified in the Methods (Eq. 7)
- Second conv uses padding=1 so the length dimension stays d=1024 (no spatial shrinkage)
- torchinfo summary parameter count matches the described configuration (two conv layers with kernels (C2,1,3) and (C,C2,3), C2 intermediate channels, C=256)
- Gradients flow through the whole module in end-to-end joint optimization with the downstream classifier

## Limitations

- Embedding dimensions and parameter optimization (embedding_channels, embedding_dim) require dataset-specific tuning; no adaptive mechanism is provided
- Effectiveness in transfer learning scenarios and transferability across datasets/tasks not yet evaluated
- Performance may be limited for severely imbalanced class distributions or too-small sample sizes

## Evidence

- [methods] applies two 1D convolutional layers with BatchNorm/ReLU/Dropout to produce a multi-channel embedding C2, and concatenates E' with C2 along the channel dimension: [evidence span withheld — anchor status: not_found]
- [methods] Apply a first 1D convolution with kernel shape (C2,1,3) (out_channels=C2, in_channels=1, kernel_size=3), followed by BatchNorm, ReLU, and Dropout: [evidence span withheld — anchor status: not_found]
- [readme] we implement a feature fusion mechanism by concatenating the initial encoded representation with the multi-channel embeddings along the channel dimension: "we implement a feature fusion mechanism by concatenating the initial encoded representation with the multi-channel embeddings along the channel dimension"
- [readme] embedding_channels | Number of embedding channels (default: `256`): "embedding_channels | Number of embedding channels (default: `256`)"
