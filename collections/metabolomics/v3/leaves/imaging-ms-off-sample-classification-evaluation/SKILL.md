---
name: imaging-ms-off-sample-classification-evaluation
description: Use when a labeled set of imaging MS ion images tagged on-/off-sample
  exists and several classifiers (deep learning, biclustering, co-localization) must
  be cross-validated and compared per-fold on F1, precision, and recall.
when_to_use_negative:
- The goal is pixel-level segmentation of off-sample regions — this skill only classifies
  whole ion images and explicitly does not solve the segmentation problem
- No expert-labeled gold standard exists and none can be curated, since the baselines
  and evaluation require on-/off-sample ground truth tags
- Data are raw instrument files rather than METASPACE-annotated ion images, since
  the biclustering baseline depends on the pixel-annotation export
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: fastai
  role: Fine-tuning the ResNet-50 deep learning classifier with default data augmentation
    and grouped five-fold cross-validation splits
  repo: https://github.com/metaspace2020/offsample
- name: PyTorch
  role: Underlying framework (v1.0) for training the ResNet-50 CNN
- name: torchvision
  role: Source of the ImageNet-pretrained ResNet-50 model (v0.2.1)
- name: scikit-learn
  role: SpectralCoclustering (v0.19.1) for the semi-automated spatio-molecular biclustering
    baseline
- name: METASPACE
  role: Source of the public datasets, ion images, and pixel-annotation export used
    for the gold standard and baselines
  repo: https://github.com/metaspace2020/metaspace/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:bc44226a0b7ca0199411a6b0db329f9a3f3e5f5f674afecda010c27aca97c4d8
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# imaging-ms-off-sample-classification-evaluation

## Summary

Reproduce the Table 1 benchmark comparing residual deep learning, semi-automated spatio-molecular biclustering, and molecular co-localization for recognizing off-sample ion images, evaluating each with grouped five-fold cross-validation on the 23,238-image expert-tagged gold standard and reporting off-sample/on-sample F1-score, precision, and recall.

## When to use

Use when a labeled set of imaging MS ion images tagged on-/off-sample exists and several classifiers (deep learning, biclustering, co-localization) must be cross-validated and compared per-fold on F1, precision, and recall.

## When NOT to use

- The goal is pixel-level segmentation of off-sample regions — this skill only classifies whole ion images and explicitly does not solve the segmentation problem
- No expert-labeled gold standard exists and none can be curated, since the baselines and evaluation require on-/off-sample ground truth tags
- Data are raw instrument files rather than METASPACE-annotated ion images, since the biclustering baseline depends on the pixel-annotation export

## Inputs

- Gold standard ion images archive (GS.tar.gz, 23,238 expert-tagged images from 87 public METASPACE datasets)
- METASPACE knowledge base pixel-annotation export (pixel-annot-export-v0.10.tar.gz)
- METASPACE dataset IDs used for grouped fold assignment

## Outputs

- cv_metrics.csv with per-fold and mean +/- standard deviation off-sample/on-sample F1-score, precision, and recall per classifier
- Trained ResNet-50 off-sample classifier (and gs_predictions.tar.gz-style predictions with predicted class and probabilities)

## How to apply

Download the gold standard ion images (GS.tar.gz from the metaspace2020/offsample releases) and the METASPACE pixel-annotation export (pixel-annot-export-v0.10.tar.gz). Preprocess ion images for deep learning by padding, converting to RGB, normalizing with ImageNet per-channel statistics, and resizing/cropping to 224x224 with fast.ai default augmentation; fine-tune a ResNet-50 (fastai v1.0.34, PyTorch v1.0, torchvision v0.2.1) with grouped five-fold cross-validation so ion images from the same METASPACE dataset stay entirely in either train or validation folds. Implement the biclustering baseline with SpectralCoclustering from scikit-learn v0.19.1 on the pixel-annotation export, plus the unsupervised biclustering and molecular co-localization baselines. Evaluate all four classifiers on each held-out fold, compute off-sample and on-sample F1-score, precision, and recall per fold, and aggregate into mean +/- standard deviation per classifier saved as cv_metrics.csv. Accept the reproduction if scores match the reported values — deep learning off-sample F1 0.97 +/- 0.01, biclustering 0.96, co-localization 0.90 — within reported standard deviations.

## Related tools

- **fastai** (Fine-tuning the ResNet-50 deep learning classifier with default data augmentation and grouped five-fold cross-validation splits) — https://github.com/metaspace2020/offsample
- **PyTorch** (Underlying framework (v1.0) for training the ResNet-50 CNN)
- **torchvision** (Source of the ImageNet-pretrained ResNet-50 model (v0.2.1))
- **scikit-learn** (SpectralCoclustering (v0.19.1) for the semi-automated spatio-molecular biclustering baseline)
- **METASPACE** (Source of the public datasets, ion images, and pixel-annotation export used for the gold standard and baselines) — https://github.com/metaspace2020/metaspace/

## Examples

```
wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz && python CNN/train.py --arch resnet50 --cv grouped-5fold
```

## Evaluation signals

- Reproduced deep learning off-sample F1 = 0.97 +/- 0.01 within the reported standard deviation
- Semi-automated spatio-molecular biclustering off-sample F1 = 0.96 and molecular co-localization F1 = 0.90 match Table 1 values
- Per-fold scores aggregated into mean +/- standard deviation rows per classifier in cv_metrics.csv with no ion images from the same dataset split across train and validation folds
- Both off-sample and on-sample F1-score, precision, and recall computed for all four classifiers on each held-out fold
- Gold standard size check: 23,238 expert-tagged ion images from 87 public METASPACE datasets loaded from GS.tar.gz

## Limitations

- Deep learning training is slow — days on a GPU-equipped workstation — which hinders optimization of the classifier
- The approach does not solve the segmentation problem of finding pixels which are in the off-sample region
- Deep learning training details are deferred to Supplementary Data D1 rather than the main text
- Ground truth about molecular content is lacking for imaging MS samples, so off-sample tags rely on expert judgements

## Evidence

- [abstract] Residual deep learning reproduced expert judgements with high agreement (off-sample F1-score 0.97 +/- 0.01): "residual deep learning (F1-score 0.97), semi-automated spatio-molecular biclustering (F1-score 0.96), and molecular co-localization (F1-score 0.90)"
- [discussion] Grouped cross-validation scheme keeps images from the same dataset together: "we used a grouped five-fold cross validation where ion images from the same dataset were all assigned either to the training or validation part"
- [discussion] Image preprocessing for deep learning uses ImageNet normalization and 224x224 resizing: "the images were normalized with the ImageNet statistics (per channel mean and std), resized and cropped to 224 × 224"
- [discussion] Biclustering baseline implemented with scikit-learn SpectralCoclustering v0.19.1: "We performed biclustering using SpectralCoclustering from the scikit-learn v0.19.1."
- [readme] README download instructions for the gold standard images: "wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz"
- [abstract] Gold standard size and provenance from 87 public METASPACE datasets: "we created a high-quality gold standard of 23,238 expert-tagged ion images from 87 public datasets from the METASPACE knowledge base"
