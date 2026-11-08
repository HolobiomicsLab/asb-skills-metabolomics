---
name: grouped-cross-validation-of-image-classifiers
description: Use when validating image classifiers on expert-tagged ion images drawn
  from multiple datasets, where images from the same source dataset must stay in a
  single fold to avoid leakage.
when_to_use_negative:
- When the goal is pixel-level segmentation of off-sample regions — the article states
  'our approach does not solve the segmentation problem of finding pixels which are
  in the off-sample region'
- When ion images cannot be mapped to a source dataset, since the grouping key (dataset
  identity) is what prevents leakage
- When evaluating on a single-dataset collection only — group structure is absent
  and plain cross-validation suffices
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3382
tools:
- name: fastai
  role: v1.0.34 library used to fine-tune the ImageNet-pretrained ResNet-50 within
    each grouped cross-validation fold, with default data augmentation
  repo: https://github.com/metaspace2020/offsample
- name: PyTorch
  role: v1.0 deep learning framework underlying the fastai classifier training and
    evaluation loops
  repo: https://github.com/metaspace2020/offsample
- name: torchvision
  role: v0.2.1 source of the ResNet-50 model architecture used for the deep learning
    fold evaluation
  repo: https://github.com/metaspace2020/offsample
- name: scikit-learn
  role: v0.19.1 SpectralCoclustering used to implement the semi-automated spatio-molecular
    biclustering baseline evaluated under the same grouped folds
  repo: https://github.com/metaspace2020/offsample
- name: METASPACE
  role: source of the public datasets and pixel-annotation export from which the gold
    standard and baselines are built
  repo: https://github.com/metaspace2020/metaspace/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:27b14fadf869d83add9e786af927b5ef92b98627678813c7ba014b16f0db8dab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/offsample
  tools:
  - Fastai
  - METASPACE
  - PyTorch
  - fastai
  - scikit-learn
  - torchvision
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# grouped-cross-validation-of-image-classifiers

## Summary

Evaluate off-sample ion image classifiers (deep learning, biclustering, co-localization) using grouped five-fold cross-validation in which all ion images from the same METASPACE dataset are kept together in either training or validation folds, preventing dataset-level leakage of the 23,238-image gold standard.

## When to use

Use when validating image classifiers on expert-tagged ion images drawn from multiple datasets, where images from the same source dataset must stay in a single fold to avoid leakage.

## When NOT to use

- When the goal is pixel-level segmentation of off-sample regions — the article states 'our approach does not solve the segmentation problem of finding pixels which are in the off-sample region'
- When ion images cannot be mapped to a source dataset, since the grouping key (dataset identity) is what prevents leakage
- When evaluating on a single-dataset collection only — group structure is absent and plain cross-validation suffices

## Inputs

- Gold standard ion images archive (GS.tar.gz from metaspace2020/offsample releases, 23,238 expert-tagged images from 87 public METASPACE datasets)
- METASPACE knowledge base pixel-annotation export (pixel-annot-export-v0.10.tar.gz) for biclustering and co-localization baselines
- Dataset-identity labels mapping each ion image to its source METASPACE dataset (used as grouping key)

## Outputs

- Per-fold off-sample and on-sample F1-score, precision, and recall for each classifier
- Aggregated mean +/- standard deviation metrics per classifier (e.g. deep learning off-sample F1 = 0.97 +/- 0.01, biclustering 0.96, co-localization 0.90)
- cv_metrics.csv containing the full cross-validation metrics table

## How to apply

Split the 23,238-image gold standard into five folds with grouping by source METASPACE dataset, so all ion images from one dataset are assigned entirely to the training or the validation part of each split. Train the ResNet-50 classifier (fastai v1.0.34, PyTorch v1.0, ImageNet-pretrained) on the training folds after preprocessing (pad, RGB conversion, ImageNet per-channel normalization, resize/crop to 224x224, fast.ai default augmentation), and fit the scikit-learn v0.19.1 SpectralCoclustering biclustering and molecular co-localization baselines on the pixel-annotation export. On each held-out fold, compute off-sample and on-sample F1-score, precision, and recall per fold. Aggregate per-fold scores into mean +/- standard deviation per classifier and save the full table as cv_metrics.csv. The rationale for grouping is that images from the same dataset share instrument and sample-preparation characteristics, so random splitting would inflate validation scores.

## Related tools

- **fastai** (v1.0.34 library used to fine-tune the ImageNet-pretrained ResNet-50 within each grouped cross-validation fold, with default data augmentation) — https://github.com/metaspace2020/offsample
- **PyTorch** (v1.0 deep learning framework underlying the fastai classifier training and evaluation loops) — https://github.com/metaspace2020/offsample
- **torchvision** (v0.2.1 source of the ResNet-50 model architecture used for the deep learning fold evaluation) — https://github.com/metaspace2020/offsample
- **scikit-learn** (v0.19.1 SpectralCoclustering used to implement the semi-automated spatio-molecular biclustering baseline evaluated under the same grouped folds) — https://github.com/metaspace2020/offsample
- **METASPACE** (source of the public datasets and pixel-annotation export from which the gold standard and baselines are built) — https://github.com/metaspace2020/metaspace/

## Examples

```
wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz  # then run grouped 5-fold CV: from sklearn.model_selection import GroupKFold; cv = GroupKFold(n_splits=5)
```

## Evaluation signals

- Reproduced deep learning off-sample F1 matches 0.97 +/- 0.01 within the reported standard deviation
- Biclustering off-sample F1 matches 0.96 and molecular co-localization matches 0.90 within reported variability
- No fold contains ion images from the same METASPACE dataset in both training and validation partitions (grouping invariant)
- Per-fold metrics for all four classifiers on the held-out folds are aggregated and persisted in cv_metrics.csv
- F1 on-sample scores for spatio-molecular biclustering approximately match or exceed the deep learning baseline, as reported

## Limitations

- Deep learning networks require substantial training time — days on a dedicated GPU-equipped workstation — which hinders optimization across folds
- The evaluation addresses image-level off-sample recognition, not pixel-level segmentation of off-sample regions
- Detailed deep learning training hyperparameters are deferred to Supplementary Data D1 rather than the main text
- Results depend on tags from the curated gold standard; ground truth about molecular content of the samples is otherwise lacking

## Evidence

- [discussion] Grouped five-fold cross-validation of classifiers: "we used a grouped five-fold cross validation where ion images from the same dataset were all assigned either to the training or validation part."
- [discussion] Image preprocessing for deep learning: "the images were normalized with the ImageNet statistics (per channel mean and std), resized and cropped to 224 × 224"
- [discussion] Libraries for training and baselines: "We used the fastai library (v1.0.34) with the PyTorch framework (v1.0)"
- [discussion] Biclustering baseline library: "We performed biclustering using SpectralCoclustering from the scikit-learn v0.19.1."
- [abstract] Deep learning performance result: "residual deep learning (F1-score 0.97)"
