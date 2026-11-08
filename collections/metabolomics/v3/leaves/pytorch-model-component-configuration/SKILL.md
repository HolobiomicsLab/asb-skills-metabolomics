---
name: pytorch-model-component-configuration
description: 'Use when replicating or extending the MSMCE representation module in
  PyTorch: wiring the LayerNorm/ReLU/Dropout Encoder, the kernel_size-3 1D-conv Channel
  Embedding, and channel-concatenation fusion onto a ResNet50 backbone.'
when_to_use_negative:
- Input is already a processed feature table or non-spectrum data — MSMCE operates
  on raw one-dimensional MS vectors represented as multi-channel embedded representations
- Analysis goal is statistical significance testing across folds (e.g. paired Wilcoxon
  signed-rank test, N=6) rather than configuring and running the module/ablation variants
- Severely imbalanced class distributions or very small sample sizes — the authors
  note deep models may fail to learn embedding representations in such extreme cases
edam_topics:
- http://edamontology.org/topic_3485
- http://edamontology.org/topic_0092
tools:
- name: torchinfo
  role: Runs the summary function on each configured MSMCE/ResNet50 variant to report
    estimated parameters and peak memory footprint
- name: thop
  role: Runs the profile function on each configured model variant to compute FLOPs,
    tabulated against baselines in the supplementary comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:da7aae352ea625a72198bd2108d5aaa5fd11ff7cea410d91d77dddf03e01b6e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - thop
  - torchinfo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3485
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pytorch-model-component-configuration

## Summary

Reproducibly instantiates the MSMCE multi-channel embedding module and its ablation variants (Encoder, Channel Embedding, Channel Concatenation) in PyTorch with the paper's exact hyperparameters, validating the configured architectures with torchinfo summaries and thop FLOPs profiling.

## When to use

Use when replicating or extending the MSMCE representation module in PyTorch: wiring the LayerNorm/ReLU/Dropout Encoder, the kernel_size-3 1D-conv Channel Embedding, and channel-concatenation fusion onto a ResNet50 backbone.

## When NOT to use

- Input is already a processed feature table or non-spectrum data — MSMCE operates on raw one-dimensional MS vectors represented as multi-channel embedded representations
- Analysis goal is statistical significance testing across folds (e.g. paired Wilcoxon signed-rank test, N=6) rather than configuring and running the module/ablation variants
- Severely imbalanced class distributions or very small sample sizes — the authors note deep models may fail to learn embedding representations in such extreme cases

## Inputs

- TIC-normalized raw MS spectrum vectors placed in datasets/ (e.g. canine_sarcoma_posion, nsclc, crlm, rcc_posion; spectrum_dim 15000/12000/9900, positive ion mode)
- Model configuration arguments (model_name such as MultiChannelEmbeddingResNet50, embedding_channels, embedding_dim, num_classes, batch_size, epochs, random_seed default 3407)

## Outputs

- Trained PyTorch model checkpoints for each ablation variant (Baseline, +Encoder, +Channel Embedding, full MSMCE)
- Per-variant Accuracy and macro-averaged F1-Score on the independent hold-out test set (compiled e.g. as ablation_results.csv)
- FLOPs and peak GPU memory estimates per variant from thop profile and torchinfo summary

## How to apply

Build the model as progressive ablation variants on a ResNet50 classifier: (1) Baseline+Encoder — a two-layer fully connected network with LayerNorm, ReLU, and Dropout; (2) Baseline+Encoder+Channel Embedding — two stacked 1D conv layers with kernel_size 3, batch normalization, ReLU, and Dropout; (3) full MSMCE — additionally concatenating the global embedding E' with the multi-channel embedding C2 along the channel dimension. Configure the module at 256 embedding channels and embedding dimension 1024, with in_channels=1 and spectrum_dim matching the dataset (15000 for Canine Sarcoma 12-class, 12000 for NSCLC/CRLM, 9900 for RCC positive ion mode) on TIC-normalized inputs. For the ablation comparison, train each variant on the entire training set (no k-fold splitting) for up to 64 epochs with Adam (lr 1e-3, weight decay 1e-5), class-weighted cross-entropy, ReduceLROnPlateau (patience 5, factor 0.1), and early stopping on validation loss (patience 10). Verify each configured architecture by running torchinfo's summary function for parameter/memory estimates and thop's profile function for FLOPs, then evaluate on an independent hold-out test set with Accuracy and macro-averaged F1.

## Related tools

- **torchinfo** (Runs the summary function on each configured MSMCE/ResNet50 variant to report estimated parameters and peak memory footprint)
- **thop** (Runs the profile function on each configured model variant to compute FLOPs, tabulated against baselines in the supplementary comparison)

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset canine_sarcoma_posion --num_classes 12 --in_channels 1 --spectrum_dim 15000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 32 --epochs 64 --use_normalization
```

## Evaluation signals

- torchinfo summary of the configured module reports 256 embedding channels and embedding dimension 1024 with in_channels=1
- Ablation table shows progressive gains: Channel Embedding variant reaches approximately Accuracy 0.8834 / F1 0.8888 and full MSMCE surpasses it (paper reports Accuracy 0.9043 for MSMCE-ResNet50 on Canine Sarcoma)
- Concatenation output dimensionality equals global embedding E' plus multi-channel embedding C2 stacked along the channel dimension
- thop-computed FLOPs of MSMCE-enhanced models match the supplementary tabulation of FLOPs and peak GPU memory (MB) versus baselines
- Training log shows early stopping patience 10, ReduceLROnPlateau schedule (patience 5, decay 0.1), and epochs capped at 64 as specified

## Limitations

- The choice of embedding dimensions and parameter optimization still require dataset-specific tuning; no adaptive feature selection mechanism exists
- Performance may be limited in extreme cases such as severely imbalanced class distributions, and deep models may fail to learn embedding representations when sample size is too small
- No direct biological interpretation of learned features (which m/z regions contribute to classification) is provided

## Evidence

- [abstract] feature fusion by channel concatenation: "we implement a feature fusion mechanism by concatenating the initial encoded representation with the multi-channel embeddings along the channel dimension"
- [results] thop used for FLOPs profiling: "FLOPs were calculated using the profile function from the Python library thop."
- [results] torchinfo used for architecture summaries: "estimated to use the summary function from the Python library torchinfo."
- [results] ablation training protocol: "model variants were trained on the entire training set (not subjected to k-fold splitting) and their performance was subsequently evaluated on the independent hold-out test set"
- [readme] TIC-normalized inputs required: "The required data files should be put into `datasets/` folder. Note that the input of each dataset is tic normalized in this implementation."
- [readme] module hyperparameters from README: "embedding_channels | Number of embedding channels (default: `256`)."
