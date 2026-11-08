---
name: fully-connected-encoder-with-layernorm-dropout
description: Use when raw one-dimensional mass spectrometry vectors (e.g. spectrum_dim
  12000/9900/15000 after 0.1 Da binning and TIC normalization) must be compressed
  to a dense global embedding before multi-channel convolutional re-encoding.
when_to_use_negative:
- When the dataset is severely class-imbalanced or the sample size is too small —
  the article notes deep models may fail to learn embedding representations in these
  extreme cases
- When the analysis requires direct biological/clinical interpretability of which
  m/z regions contribute to classification — the authors state this was not investigated
  and interpretability methods for such embeddings are lacking
- When the input is already a multi-channel or image-like representation (e.g. converted
  spectra) rather than raw 1D MS vectors, since this encoder is designed to compress
  D-dimensional 1D spectra
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: PyTorch
  role: Framework for implementing the two-layer Linear→LayerNorm→ReLU→Dropout encoder
    as a single nn.Module with joint end-to-end gradient flow
  repo: https://github.com/WoodFY/MSMCE
- name: torchinfo
  role: summary() function used to confirm layer configuration and estimate model
    size/parameters of the encoder within the MSMCE module
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:4e6162cf5ff386124ddc323941e63b0dd31894273098c40287f02936db169ca5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - PyTorch
  - torchinfo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# fully-connected-encoder-with-layernorm-dropout

## Summary

A two-layer fully connected encoder (Linear D→2048 → LayerNorm → ReLU → Dropout → Linear 2048→1024) that compresses a batch of TIC-normalized, 0.1 Da-binned one-dimensional MS vectors into a global embedding E of shape B×1024, serving as the first stage of the MSMCE representation module (Zhang et al., PLOS One 2025).

## When to use

Use when raw one-dimensional mass spectrometry vectors (e.g. spectrum_dim 12000/9900/15000 after 0.1 Da binning and TIC normalization) must be compressed to a dense global embedding before multi-channel convolutional re-encoding.

## When NOT to use

- When the dataset is severely class-imbalanced or the sample size is too small — the article notes deep models may fail to learn embedding representations in these extreme cases
- When the analysis requires direct biological/clinical interpretability of which m/z regions contribute to classification — the authors state this was not investigated and interpretability methods for such embeddings are lacking
- When the input is already a multi-channel or image-like representation (e.g. converted spectra) rather than raw 1D MS vectors, since this encoder is designed to compress D-dimensional 1D spectra

## Inputs

- Batch of 1D mass spectrometry vectors X of shape B×D, TIC-normalized and m/z-binned at 0.1 Da (e.g. D=12000 for NSCLC, 9900 for RCC, 15000 for Canine Sarcoma)
- Configuration parameters: spectrum_dim (input D), embedding_dim d=1024, embedding_channels C=256, hidden width 2048

## Outputs

- Global embedding E of shape B×1024 (dense feature vector per MS sample)
- Within MSMCE: fused tensor O of shape B×(1+C)×d, i.e. B×257×1024, after reshaping E to B×1×d and concatenating with the multi-channel embedding C2

## How to apply

Load a batch X of 1D MS vectors with shape B×D from the preprocessed dataset files (TIC-normalized, m/z-binned at 0.1 Da). Apply a linear layer mapping D (e.g. 12000 for NSCLC) to 2048 (W1 ∈ R^(D×2048)), then LayerNorm, ReLU activation, and Dropout, followed by a second linear layer mapping 2048 to the embedding dimension d=1024 (E = H1·W2 + b2). The LayerNorm/ReLU/Dropout ordering stabilizes training of the jointly optimized end-to-end framework, since this encoder feeds downstream classifiers (ResNet50/DenseNet121/EfficientNetB0/LSTM/Transformer) that are trained together with the representation module. In the full MSMCE pipeline, the resulting embedding E is reshaped to B×1×d and concatenated with a convolutionally derived multi-channel embedding C2 (C=256 channels) to yield O ∈ R^(B×257×1024). Use PyTorch nn.Module wrapping so gradients flow jointly, and inspect the architecture with torchinfo to confirm layer configuration and parameter counts.

## Related tools

- **PyTorch** (Framework for implementing the two-layer Linear→LayerNorm→ReLU→Dropout encoder as a single nn.Module with joint end-to-end gradient flow) — https://github.com/WoodFY/MSMCE
- **torchinfo** (summary() function used to confirm layer configuration and estimate model size/parameters of the encoder within the MSMCE module)

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 64 --epochs 64 --use_normalization
```

## Evaluation signals

- Forward pass on a sample batch runs without error
- Output tensor O has the exact shape B×257×1024 (1 global channel + C=256 conv channels, d=1024) as specified in the Methods (Eq. 7)
- Layer order inside the encoder is Linear(D→2048) → LayerNorm → ReLU → Dropout → Linear(2048→1024), matching the described configuration
- Total learnable parameter count is recorded via torchinfo summary and matches the intended architecture
- Gradients flow through encoder, channel embedding, and downstream classifier during joint end-to-end training (no detached/re-frozen components)

## Limitations

- Embedding dimensions (d=1024) and parameter optimization require dataset-specific tuning; no adaptive mechanism exists
- Performance may be limited for severely imbalanced class distributions or very small sample sizes
- Lack of biological/clinical interpretability of the learned embedding features (which m/z regions contribute to classification) is an open limitation

## Evidence

- [methods] Two-layer FC encoder with LayerNorm, ReLU, Dropout: [evidence span withheld — anchor status: not_found]
- [abstract] Concatenation-based feature fusion in MSMCE: "we implement a feature fusion mechanism by concatenating the initial encoded representation with the multi-channel embeddings along the channel dimension"
- [readme] TIC-normalized input data requirement: "The required data files should be put into `datasets/` folder. Note that the input of each dataset is tic normalized in this implementation."
