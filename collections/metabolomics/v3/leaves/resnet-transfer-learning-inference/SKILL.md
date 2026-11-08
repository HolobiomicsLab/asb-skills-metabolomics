---
name: resnet-transfer-learning-inference
description: Use when ion images from METASPACE-style imaging MS datasets must be
  classified as off-sample vs on-sample with a pretrained ResNet-50, or when reproducing
  the paper's off-sample frequency counts over DESI datasets.
when_to_use_negative:
- The goal is per-pixel localization of the off-sample region — the authors state
  'our approach does not solve the segmentation problem of finding pixels which are
  in the off-sample region'
- Input is raw imaging MS data (e.g. imzML/mzML) rather than pre-generated ion images
  — the model operates on rendered ion images, not raw spectra
- Labelled training data is absent and only unsupervised/semi-automated approaches
  are acceptable — biclustering (F1 0.96) or molecular co-localization (F1 0.90) are
  the paper's alternatives, but this skill specifically requires a trained supervised
  model
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: fastai
  role: Training and inference framework for the CNN classifier, v1.0.34, with its
    default data augmentation settings
  repo: https://github.com/metaspace2020/offsample
- name: PyTorch
  role: Deep learning framework (v1.0) underlying fastai on which the ResNet-50 model
    runs
  repo: https://github.com/metaspace2020/offsample
- name: torchvision
  role: Source of the ResNet-50 model pretrained on ImageNet (v0.2.1) used for transfer
    learning
  repo: https://github.com/metaspace2020/offsample
- name: METASPACE
  role: Source of the public imaging MS datasets and ion images from the METASPACE
    knowledge base used for training and inference
  repo: http://metaspace2020.eu
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:66cad937eb170194bddacd0772306a359d822596bb81f1c68c8a1ef5c40d688b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/offsample
  tools:
  - Fastai
  - METASPACE
  - PyTorch
  - ResNet-50
  - fastai
  - torchvision
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# resnet-transfer-learning-inference

## Summary

Apply the ResNet-50 CNN (pretrained on ImageNet) via the fastai/PyTorch stack to classify MALDI imaging MS ion images as off-sample or on-sample, reproducing the OffsampleAI deep learning approach that reached F1-score 0.97 on the 23,238-image gold standard.

## When to use

Use when ion images from METASPACE-style imaging MS datasets must be classified as off-sample vs on-sample with a pretrained ResNet-50, or when reproducing the paper's off-sample frequency counts over DESI datasets.

## When NOT to use

- The goal is per-pixel localization of the off-sample region — the authors state 'our approach does not solve the segmentation problem of finding pixels which are in the off-sample region'
- Input is raw imaging MS data (e.g. imzML/mzML) rather than pre-generated ion images — the model operates on rendered ion images, not raw spectra
- Labelled training data is absent and only unsupervised/semi-automated approaches are acceptable — biclustering (F1 0.96) or molecular co-localization (F1 0.90) are the paper's alternatives, but this skill specifically requires a trained supervised model

## Inputs

- Gold standard ion images archive (GS.tar.gz, 23,238 expert-tagged images from 87 public METASPACE datasets)
- Individual MALDI/DESI ion images (grayscale intensity maps) for molecular formulae, e.g. the 353 DHB matrix-cluster formulas
- Trained ResNet-50 classifier (fastai learner / model weights from the metaspace2020/offsample release)

## Outputs

- Per-image predicted class (off-sample / on-sample) and predicted probability
- Images grouped by predicted class with predicted probabilities (cf. gs_predictions.tar.gz)
- Per-dataset off-sample frequency counts per molecular formula (e.g. C18H30O3S: 14/24, C17H28O3S: 13/24)

## How to apply

Load the ResNet-50 model pretrained on ImageNet from torchvision (v0.2.1) within the fastai (v1.0.34) / PyTorch (v1.0) framework as used by the authors. Preprocess each gold standard ion image: convert to RGB, zero-pad as needed, normalize with the ImageNet per-channel mean and std statistics, then resize and crop to 224x224; during training, apply fast.ai default augmentation (crop/pad, flips, warping, rotation, zooming, brightness/contrast). Run batch inference over the ion images (e.g. the images unpacked from GS.tar.gz, release 0.2 of metaspace2020/offsample) to obtain a predicted class and predicted probability per image. Aggregate predictions per molecular formula per dataset using the 353 DHB matrix-cluster molecular formulas, and use grouped five-fold cross-validation (dataset-level splits) when evaluating, since ion images from the same dataset must not straddle train/validation sets. Validate against the published result that deep learning achieved F1-score 0.97 and that C18H30O3S is flagged off-sample in 14/24 DESI datasets and C17H28O3S in 13/24, matching Supplementary Data D5.

## Related tools

- **fastai** (Training and inference framework for the CNN classifier, v1.0.34, with its default data augmentation settings) — https://github.com/metaspace2020/offsample
- **PyTorch** (Deep learning framework (v1.0) underlying fastai on which the ResNet-50 model runs) — https://github.com/metaspace2020/offsample
- **torchvision** (Source of the ResNet-50 model pretrained on ImageNet (v0.2.1) used for transfer learning) — https://github.com/metaspace2020/offsample
- **METASPACE** (Source of the public imaging MS datasets and ion images from the METASPACE knowledge base used for training and inference) — http://metaspace2020.eu

## Examples

```
wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz
```

## Evaluation signals

- Reproduce the abstract result: residual deep learning achieves F1-score 0.97 on the gold standard under grouped five-fold cross-validation (images from the same dataset never split across train/validation)
- Predictions match the released gs_predictions.tar.gz grouping: every gold standard image gets a predicted class and probability consistent with the archive
- Off-sample frequency counts match Supplementary Data D5: C18H30O3S off-sample in 14/24 DESI datasets and C17H28O3S in 13/24
- Preprocessing invariants: every image is RGB, normalized with ImageNet per-channel mean/std, and exactly 224x224 before entering the network
- Formula aggregation uses exactly the 353 DHB matrix-cluster molecular formulas from DHB_mc_mol_formulas_predicted.csv

## Limitations

- Deep learning training requires substantial time — 'namely days on specially-devoted GPU... which hinders optimization'
- The classifier assigns a label to a whole ion image; it does not perform segmentation of off-sample pixels
- Performance is characterized on DHB matrix-cluster off-sample images and the 87-dataset METASPACE gold standard; behavior on other matrices/labs may differ

## Evidence

- [discussion] ImageNet normalization and 224x224 preprocessing: "the images were normalized with the ImageNet statistics (per channel mean and std), resized and cropped to 224 × 224"
- [discussion] fastai/PyTorch framework: "We used the fastai library (v1.0.34) with the PyTorch framework (v1.0)"
- [discussion] ResNet-50 from torchvision: "with the ResNet-50 model from the torchvision library (v0.2.1)"
- [abstract] Deep learning F1-score: "residual deep learning (F1-score 0.97)"
- [readme] README on pretrained ResNet-50: "We trained Convolutional Neural Networks using Fastai and PyTorch libraries. The best performance we achieved using Resnet50 CNN pretrained on Imagenet."
- [methods] Gold standard data download: "wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz"
