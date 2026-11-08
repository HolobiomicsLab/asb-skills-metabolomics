---
name: cnn-transfer-learning-fine-tuning
description: Use when ion images (2D spatial intensity maps) from imaging MS must
  be automatically classified as off-sample/on-sample, and a small curated labeled
  set exists that is too large to tag manually but suitable for fine-tuning a pretrained
  vision model.
when_to_use_negative:
- The goal is pixel-level segmentation of off-sample regions within an image — the
  paper states 'our approach does not solve the segmentation problem of finding pixels
  which are in the off-sample region'
- 'Training-time budget is limited: the deep network requires days on a GPU-equipped
  power station, so semi-automated biclustering (F1 0.96) or molecular co-localization
  (F1 0.90) may be preferable'
- No labeled gold standard exists for the domain; transfer learning still requires
  expert-tagged images for fine-tuning and validation
edam_operation: http://edamontology.org/operation_0399
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: fastai
  role: Library (v1.0.34) used to fine-tune the CNN with its default data augmentation
    settings
  repo: https://github.com/metaspace2020/offsample
- name: PyTorch
  role: Underlying deep learning framework (v1.0) for the fastai training loop
  repo: https://github.com/metaspace2020/offsample
- name: torchvision
  role: Source of the ResNet-50 model pretrained on ImageNet (v0.2.1)
  repo: https://github.com/metaspace2020/offsample
- name: METASPACE
  role: Source of the 87 public datasets and 23,238 expert-tagged ion images used
    for training/evaluation; the trained classifier was integrated as an off-sample
    filter
  repo: http://metaspace2020.eu
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:316058aa1614238f4312ee546451a03c41eaad0912c27fa12ca7a37852055a46
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
  edam_operation: http://edamontology.org/operation_0399
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# CNN transfer learning fine-tuning

## Summary

Fine-tunes a ResNet-50 CNN pretrained on ImageNet with fastai/PyTorch to classify ion images from imaging mass spectrometry as off-sample vs on-sample, reproducing expert tagger judgements with off-sample F1 = 0.97 ± 0.01 on a 23,238-image gold standard.

## When to use

Use when ion images (2D spatial intensity maps) from imaging MS must be automatically classified as off-sample/on-sample, and a small curated labeled set exists that is too large to tag manually but suitable for fine-tuning a pretrained vision model.

## When NOT to use

- The goal is pixel-level segmentation of off-sample regions within an image — the paper states 'our approach does not solve the segmentation problem of finding pixels which are in the off-sample region'
- Training-time budget is limited: the deep network requires days on a GPU-equipped power station, so semi-automated biclustering (F1 0.96) or molecular co-localization (F1 0.90) may be preferable
- No labeled gold standard exists for the domain; transfer learning still requires expert-tagged images for fine-tuning and validation

## Inputs

- Gold standard ion images archive (GS.tar.gz from github.com/metaspace2020/offsample releases), 23,238 expert-tagged images from 87 public METASPACE datasets
- Per-image labels: off-sample / on-sample (unknown excluded)
- Dataset identifiers per ion image (for grouped fold assignment)

## Outputs

- Trained ResNet-50 classifier (released as a microservice/REST API in metaspace2020/metaspace off-sample service)
- Per-fold off-sample and on-sample F1-score, precision, and recall aggregated as mean ± standard deviation
- Predicted classes and probabilities for gold standard images (gs_predictions.tar.gz)

## How to apply

Preprocess each ion image by zero-padding to square, converting to RGB, normalizing with ImageNet per-channel mean/std, and resizing/crop to 224×224; apply fast.ai default data augmentation (flips, warping, rotation, zooming, brightness/contrast). Fine-tune a ResNet-50 pretrained on ImageNet using fastai v1.0.34 on PyTorch v1.0 with torchvision v0.2.1. Critically, use grouped five-fold cross-validation in which all ion images from the same METASPACE dataset are assigned entirely to training or validation folds, preventing dataset-level leakage. Evaluate off-sample and on-sample F1-score, precision, and recall per fold, then report mean ± standard deviation across folds. Detailed training hyperparameters are deferred to Supplementary Data D1 of the paper.

## Related tools

- **fastai** (Library (v1.0.34) used to fine-tune the CNN with its default data augmentation settings) — https://github.com/metaspace2020/offsample
- **PyTorch** (Underlying deep learning framework (v1.0) for the fastai training loop) — https://github.com/metaspace2020/offsample
- **torchvision** (Source of the ResNet-50 model pretrained on ImageNet (v0.2.1)) — https://github.com/metaspace2020/offsample
- **METASPACE** (Source of the 87 public datasets and 23,238 expert-tagged ion images used for training/evaluation; the trained classifier was integrated as an off-sample filter) — http://metaspace2020.eu

## Evaluation signals

- Reproduced off-sample F1-score matches 0.97 ± 0.01 under the grouped five-fold cross-validation protocol
- Grouped fold integrity check: no METASPACE dataset contributes ion images to both training and validation folds of the same split
- All images are exactly 224×224 RGB after preprocessing and were normalized with ImageNet per-channel statistics
- On-sample F1 for the deep model should be at or slightly below the biclustering baseline, which 'almost matches and outperforms it in the F1 on-sample score'
- Predicted class probabilities exported with gs_predictions.tar.gz group images consistently with their predicted classes

## Limitations

- Training takes days on a dedicated GPU power station, which hinders optimization
- The approach does not solve pixel-level segmentation of off-sample regions
- Deep learning training details are deferred to Supplementary Data D1 rather than the main text
- Images from the same dataset must never be split across training and validation folds, or scores will be inflated by leakage

## Evidence

- [discussion] fastai/PyTorch versions: "We used the fastai library (v1.0.34) with the PyTorch framework (v1.0)"
- [discussion] ImageNet preprocessing: "the images were normalized with the ImageNet statistics (per channel mean and std), resized and cropped to 224 × 224"
- [discussion] grouped cross-validation: "we used a grouped five-fold cross validation where ion images from the same dataset were all assigned either to the training or validation part."
- [readme] ResNet-50 pretrained on ImageNet: "The best performance we achieved using Resnet50 CNN pretrained on Imagenet."
- [abstract] off-sample F1 result: "residual deep learning (F1-score 0.97)"
