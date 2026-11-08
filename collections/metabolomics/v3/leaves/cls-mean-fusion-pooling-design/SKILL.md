---
name: cls-mean-fusion-pooling-design
description: Use when building or retraining a bi-encoder/cross-encoder hybrid for
  pairwise 1H NMR spectra classification and you must decide how to aggregate cross-encoder
  token outputs into a single feature vector for the classifier head.
when_to_use_negative:
- When the model is a single-branch CNN without a cross-encoder emitting a [CLS] token
  — there is no dual view to fuse
- When you already have a single pooled embedding from a prior aggregation step; adding
  fusion pooling on top would double-count features
- When applying pooling to non-spectral (e.g. text-only or image-only) inputs where
  the paper's NMR-specific fusion rationale does not transfer
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3047
tools:
- name: PyTorch
  role: Implements the feature-aggregation module that concatenates the cross-encoder's
    [CLS] hidden state with mean-pooled token features before the classifier head
  repo: https://github.com/yfWang01/FlavorFormer.git
- name: FlavorFormer
  role: Reference implementation whose model folder contains the fusion pooling module
    and whose demo.ipynb trains and evaluates it on NMR mixture data
  repo: https://github.com/yfWang01/FlavorFormer.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.microc.2025.115372
    title: 'FlavorFormer: hybrid deep learning for identifying compounds in flavor
      mixtures based on NMR spectroscopy'
  source_content_hash: sha256:2fd052b63d03b0bfbfc2e00399c53965fad6c9e4fc6f5c5b8f805006a73429f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yfWang01/FlavorFormer.git
  tools:
  - FlavorFormer
  - PyTorch
  - Python
  - conda
  - git
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3047
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# CLS-mean fusion pooling design

## Summary

Design a feature-aggregation (pooling) step for the FlavorFormer hybrid CNN–Transformer model that fuses the cross-encoder's learnable [CLS] token output with mean pooling over encoded sequence features, before feeding the classifier that outputs a Sigmoid-constrained presence score in [0,1].

## When to use

Use when building or retraining a bi-encoder/cross-encoder hybrid for pairwise 1H NMR spectra classification and you must decide how to aggregate cross-encoder token outputs into a single feature vector for the classifier head.

## When NOT to use

- When the model is a single-branch CNN without a cross-encoder emitting a [CLS] token — there is no dual view to fuse
- When you already have a single pooled embedding from a prior aggregation step; adding fusion pooling on top would double-count features
- When applying pooling to non-spectral (e.g. text-only or image-only) inputs where the paper's NMR-specific fusion rationale does not transfer

## Inputs

- Token-level output tensors from the Transformer cross-encoder (hidden size 64) over a pair of 1H NMR spectra
- The learnable [CLS] token's final hidden-state vector
- Encoded sequence features over which mean pooling is applied

## Outputs

- A fused feature vector combining [CLS] token output and mean-pooled sequence features
- Input feature vector consumed by the classifier head producing a Sigmoid-constrained score in [0,1]

## How to apply

After the Transformer cross-encoder (linear embedding to hidden dimension 64, positional encoding, learnable [CLS] and [SEP] tokens, four encoder layers with 4-head self-attention) produces token-wise features, extract two views of the sequence: the [CLS] token's final hidden state, which carries a learned global summary, and the arithmetic mean over the remaining encoded token features. Concatenate (or otherwise fuse) these two vectors in the feature-aggregation module so the classifier receives both a global summary and an average local representation of the spectral pair. Train the downstream classifier with BCEWithLogitsLoss using a positive-class weight of 3 so the fused features are optimized toward the Sigmoid-constrained presence score. The paper's ablation reports that fusion pooling (together with the weighted loss) significantly boosts identification accuracy, so retain it when reproducing the reported test-set ACC of 98.84 ± 0.86 %.

## Related tools

- **PyTorch** (Implements the feature-aggregation module that concatenates the cross-encoder's [CLS] hidden state with mean-pooled token features before the classifier head) — https://github.com/yfWang01/FlavorFormer.git
- **FlavorFormer** (Reference implementation whose model folder contains the fusion pooling module and whose demo.ipynb trains and evaluates it on NMR mixture data) — https://github.com/yfWang01/FlavorFormer.git

## Examples

```
cd FlavorFormer && conda env create -f environment.yml && conda activate FlavorFormer && jupyter notebook demo.ipynb
```

## Evaluation signals

- The fused feature vector dimensionality equals 2 × hidden size (i.e. 128 when hidden dimension is 64), matching the concatenation of [CLS] and mean-pooled branches
- End-to-end forward and backward passes through the fusion pooling module run without errors on sample NMR spectra from the repository's data folder
- With fusion pooling plus the weighted loss (positive weight 3), reproduced test-set metrics approach ACC 98.84 ± 0.86 %, TPR 99.90 ± 0.10 %, FPR 2.24 ± 1.77 %
- Removing fusion pooling while keeping the weighted loss yields measurably lower identification accuracy, mirroring the paper's ablation claim
- Classifier output after Sigmoid falls within [0,1] for all validation pairs

## Limitations

- Fusion pooling was validated only on augmented 1H NMR spectra and two experimental flavor-mixture datasets; behavior on other spectral types is untested
- The design assumes the cross-encoder uses learnable [CLS] and [SEP] tokens with hidden dimension 64; other Transformer configurations may require adapting the aggregation
- The accuracy boost is attributed jointly to fusion pooling and the weighted loss function, so isolating the pooling effect alone requires an ablation not fully specified in the README

## Evidence

- [methods] a feature aggregation module, and a classifier that outputs a Sigmoid-constrained score in [0,1]: [evidence span withheld — anchor status: not_found]
- [abstract] Fusion pooling and weighted loss function significantly boost identification accuracy: "Fusion pooling and weighted loss function significantly boost identification accuracy."
- [intro] it leverages a combination of a bi-encoder and cross-encoder, a fusion pooling strategy, and a weighted loss function to identify compounds correctly: "it leverages a combination of a bi-encoder and cross-encoder, a fusion pooling strategy, and a weighted loss function to identify compounds correctly."
- [methods] The model parameters is located in the model folder and the required data is in the data folder: "The model parameters is located in the [model](model/) folder and the required data is in the [data](data/) folder, FlavorFormer can be reload and predict easily."
- [methods] learnable [CLS] and [SEP] tokens, four Transformer encoder layers with 4-head multi-head self-attention: [evidence span withheld — anchor status: not_found]
