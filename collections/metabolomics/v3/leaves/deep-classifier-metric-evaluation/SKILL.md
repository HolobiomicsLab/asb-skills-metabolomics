---
name: deep-classifier-metric-evaluation
description: Use when a CNN/sequence classifier trained on TIC-normalized raw MS spectra
  (e.g. NSCLC, CRLM, RCC, Canine Sarcoma) needs Accuracy/F1 evaluation on a hold-out
  split plus statistical comparison against a baseline.
when_to_use_negative:
- Input data is a feature-selected, dimensionality-reduced table rather than TIC-normalized
  raw spectra — the MSMCE/hold-out protocol expects raw vectors with known spectrum_dim.
- The dataset is severely imbalanced or too small for deep models to learn embedding
  representations — the article explicitly flags these as cases where the MSMCE approach
  may fail.
- Only a single split without an independent hold-out set is available — the Wilcoxon
  and bootstrap comparisons require k-fold (N = 6) folds against a baseline.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: thop
  role: Profiles the trained models to compute Floating-Point Operations (FLOPs) reported
    alongside accuracy for MSMCE vs baseline variants.
  repo: https://github.com/WoodFY/MSMCE
- name: torchinfo
  role: Summarizes model parameters and estimated peak GPU memory footprint for each
    MSMCE-enhanced and baseline architecture.
  repo: https://github.com/WoodFY/MSMCE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:6f0c1ec1d8fb1a6a26aa666c12b99b44a43f5cb4084b468b636c9d0536680eca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - thop
  - torchinfo
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# deep-classifier-metric-evaluation

## Summary

Evaluates deep learning classifiers for raw mass spectrometry data by computing Accuracy and macro-unweighted-averaged F1-Score on an independent hold-out test set, and by pairing results with the Wilcoxon signed-rank test, bootstrap 95% CIs, and efficiency metrics (FLOPs/GPU memory) so that MSMCE-style representation modules can be judged fairly.

## When to use

Use when a CNN/sequence classifier trained on TIC-normalized raw MS spectra (e.g. NSCLC, CRLM, RCC, Canine Sarcoma) needs Accuracy/F1 evaluation on a hold-out split plus statistical comparison against a baseline.

## When NOT to use

- Input data is a feature-selected, dimensionality-reduced table rather than TIC-normalized raw spectra — the MSMCE/hold-out protocol expects raw vectors with known spectrum_dim.
- The dataset is severely imbalanced or too small for deep models to learn embedding representations — the article explicitly flags these as cases where the MSMCE approach may fail.
- Only a single split without an independent hold-out set is available — the Wilcoxon and bootstrap comparisons require k-fold (N = 6) folds against a baseline.

## Inputs

- TIC-normalized raw MS spectrum vectors (positive ion mode; spectrum_dim 12000, 9900 or 15000)
- Stratified file-level k-fold train/hold-out split definitions (N = 6 folds)
- Trained model checkpoints for baseline (e.g. ResNet50, DenseNet121) and MSMCE-enhanced variants

## Outputs

- Hold-out test Accuracy and macro-unweighted-averaged F1-Score per model (e.g. MSMCE-ResNet50 accuracy 0.9785 on NSCLC)
- Paired Wilcoxon signed-rank test p-values for baseline vs MSMCE-integrated models (two-sided, N = 6 folds, p < 0.05 significant)
- Bootstrap 95% confidence intervals for mean Accuracy, Precision, Recall, F1-Score across k folds
- Estimated peak GPU memory footprint (MB) and FLOPs per model variant

## How to apply

Split the TIC-normalized spectrum dataset (positive ion mode, spectrum_dim e.g. 12000 or 15000) into a stratified file-level k-fold (N = 6) train/validation scheme, reserving an independent hold-out test set. Train each variant for up to 64 epochs with Adam (lr 1e-3, weight decay 1e-5), class-weighted cross-entropy loss, ReduceLROnPlateau (patience 5, factor 0.1) and early stopping with patience 10, evaluating each epoch on validation loss. Compute Accuracy and F1-Score with macro unweighted averaging on the hold-out test set; compare a baseline (e.g. ResNet50) versus the MSMCE-enhanced model using the paired Wilcoxon signed-rank test with p < 0.05 considered significant. Report bootstrap 95% confidence intervals for mean Accuracy, Precision, Recall and F1-Score across the k folds, and supplement with FLOPs (via thop) and parameter/peak GPU memory summaries (via torchinfo) to document the computational cost of each module.

## Related tools

- **thop** (Profiles the trained models to compute Floating-Point Operations (FLOPs) reported alongside accuracy for MSMCE vs baseline variants.) — https://github.com/WoodFY/MSMCE
- **torchinfo** (Summarizes model parameters and estimated peak GPU memory footprint for each MSMCE-enhanced and baseline architecture.) — https://github.com/WoodFY/MSMCE

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 64 --epochs 64 --use_normalization
```

## Evaluation signals

- Accuracy and macro-unweighted-averaged F1-Score are reported on the independent hold-out test set, not on training/validation folds.
- Wilcoxon paired test yields p < 0.05 when comparing MSMCE-integrated model against its baseline (e.g. 0.03 for MSMCE-ResNet50 vs ResNet50 on NSCLC, N = 6).
- Bootstrap 95% CIs are provided for all four mean metrics (Accuracy, Precision, Recall, F1-Score) across the 6 folds.
- FLOPs (via thop) and parameter/memory summaries (via torchinfo) accompany accuracy results, showing MSMCE adds little computational cost.
- Known accuracy benchmarks reproduce: NSCLC 0.9785 (MSMCE-ResNet50), CRLM 6.66% accuracy gain (MSMCE-DenseNet121), RCC F1 0.6946→0.7952 (MSMCE-DenseNet121).

## Limitations

- Embedding dimensions and parameters require dataset-specific tuning; no adaptive feature selection mechanism exists.
- Performance may degrade on severely imbalanced class distributions or very small sample sizes where deep models fail to learn embeddings.
- Biological/clinical interpretability of learned embedding features (e.g. which m/z regions contribute) was not investigated, limiting clinical decision-making value.
- Transfer-learning effectiveness across datasets/tasks has not yet been evaluated.

## Evidence

- [methods] Accuracy and F1-Score macro unweighted averaging used for hold-out evaluation: [evidence span withheld — anchor status: not_found]
- [results] Wilcoxon paired test protocol: "we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines"
- [results] FLOPs measured with thop: "FLOPs were calculated using the profile function from the Python library thop."
- [results] torchinfo used for parameter summaries: "estimated to use the summary function from the Python library torchinfo."
- [results] NSCLC accuracy benchmark: "MSMCE-ResNet50 achieved an accuracy of 0.9785, a statistically significant increase of 1.28% (p = 0.03, N = 6)"
