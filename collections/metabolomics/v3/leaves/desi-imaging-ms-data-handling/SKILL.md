---
name: desi-imaging-ms-data-handling
description: Use when you need to classify ion images from multiple DESI imaging MS
  datasets as off-sample/on-sample and compute per-formula off-sample frequency counts
  to detect recurring matrix-cluster artifacts.
when_to_use_negative:
- When the goal is pixel-level segmentation of the off-sample region — the article
  states 'our approach does not solve the segmentation problem of finding pixels which
  are in the off-sample region'
- When ion images are from non-DESI modalities or you need class probabilities for
  individual pixels rather than per-image labels
- When you lack the specific gold standard archive or pretrained model from metaspace2020/offsample
  — do not substitute arbitrary classifiers without retraining
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: METASPACE
  role: Source of the public DESI imaging MS datasets and gold standard ion images
  repo: https://github.com/metaspace2020/metaspace/
- name: fastai
  role: Library (v1.0.34) used to train and run the ResNet-50 off-sample classifier
  repo: https://github.com/metaspace2020/offsample
- name: PyTorch
  role: Deep learning framework (v1.0) underlying the fastai classifier inference
  repo: https://github.com/metaspace2020/offsample
- name: torchvision
  role: Provides the ResNet-50 model architecture (v0.2.1) pretrained on ImageNet
  repo: https://github.com/metaspace2020/offsample
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:84b8f485afa8360c62b0d63128855bd045c97eece6dc093a93ee2910630ce068
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - Fastai
  - METASPACE
  - PyTorch
  - fastai
  - torchvision
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# desi-imaging-ms-data-handling

## Summary

A workflow for applying the OffsampleAI deep learning off-sample classifier to DESI imaging MS datasets from the METASPACE gold standard, and aggregating predictions per molecular formula to identify which ions (e.g., DHB matrix clusters like C18H30O3S and C17H28O3S) are systematically off-sample.

## When to use

Use when you need to classify ion images from multiple DESI imaging MS datasets as off-sample/on-sample and compute per-formula off-sample frequency counts to detect recurring matrix-cluster artifacts.

## When NOT to use

- When the goal is pixel-level segmentation of the off-sample region — the article states 'our approach does not solve the segmentation problem of finding pixels which are in the off-sample region'
- When ion images are from non-DESI modalities or you need class probabilities for individual pixels rather than per-image labels
- When you lack the specific gold standard archive or pretrained model from metaspace2020/offsample — do not substitute arbitrary classifiers without retraining

## Inputs

- Gold standard ion images archive (GS.tar.gz) from metaspace2020/offsample release 0.2
- DESI imaging MS ion images from public METASPACE datasets (24 DESI datasets from Takats lab, Imperial College London and Eberlin lab, UT Austin)
- List of 353 DHB matrix-cluster molecular formulas (DHB mc mol formulas, predicted.csv)
- Best off-sample ResNet-50 classifier trained with fastai/PyTorch

## Outputs

- Per-image off-sample/on-sample predicted class and predicted probability
- Per molecular formula per dataset off-sample predictions
- Cross-dataset off-sample frequency counts (e.g., C18H30O3S off-sample in 14/24, C17H28O3S in 13/24 DESI datasets)

## How to apply

Download and unpack the gold standard ion images archive (GS.tar.gz) from the metaspace2020/offsample GitHub release v0.2. Load a ResNet-50 CNN pretrained on ImageNet, trained via fastai v1.0.34 on the PyTorch v1.0 framework; preprocess each ion image by converting to RGB, normalizing with the ImageNet per-channel mean/std statistics, and resizing/cropping to 224x224. Run batch inference to obtain a predicted off-sample/on-sample class and probability for every image. Restrict the formula list to the 353 DHB matrix-cluster molecular formulas generated per Keller and Li (2000), aggregate predictions per molecular formula per dataset across the 24 DESI datasets, and compute off-sample frequency counts. Rationale: matrix-cluster ions such as DHB adducts consistently appear off-sample, so cross-dataset frequency reveals biologically irrelevant signals.

## Related tools

- **METASPACE** (Source of the public DESI imaging MS datasets and gold standard ion images) — https://github.com/metaspace2020/metaspace/
- **fastai** (Library (v1.0.34) used to train and run the ResNet-50 off-sample classifier) — https://github.com/metaspace2020/offsample
- **PyTorch** (Deep learning framework (v1.0) underlying the fastai classifier inference) — https://github.com/metaspace2020/offsample
- **torchvision** (Provides the ResNet-50 model architecture (v0.2.1) pretrained on ImageNet) — https://github.com/metaspace2020/offsample

## Examples

```
wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz
```

## Evaluation signals

- Reproduced frequency counts match Supplementary Data D5: C18H30O3S classified off-sample in 14 of 24 DESI datasets and C17H28O3S in 13 of 24 datasets
- Every ion image is preprocessed to RGB at 224x224 with ImageNet per-channel normalization before inference
- Aggregation is restricted to the 353 DHB matrix-cluster molecular formulas listed in the repository CSV
- Predicted probabilities for each image exist alongside predicted classes (as in gs_predictions.tar.gz grouping)

## Limitations

- The approach classifies whole ion images, not pixels: 'our approach does not solve the segmentation problem of finding pixels which are in the off-sample region'
- Deep learning networks require substantial training time ('days on specially-devoted GPU ... power station'), which hinders optimization
- Findings are specific to DHB matrix-cluster off-sample signals in the gold standard DESI datasets; generalization to other matrices or labs is not established here

## Evidence

- [readme] Gold standard ion images archive download: "wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz"
- [discussion] Image preprocessing for the CNN: "the images were normalized with the ImageNet statistics (per channel mean and std), resized and cropped to 224 × 224"
- [discussion] Classifier training framework: "We used the fastai library (v1.0.34) with the PyTorch framework (v1.0)"
- [discussion] ResNet-50 architecture source: "with the ResNet-50 model from the torchvision library (v0.2.1)"
