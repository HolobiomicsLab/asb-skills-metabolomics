---
name: ion-image-preprocessing-for-cnn-inference
description: Use when ion images (e.g. from METASPACE gold standard GS.tar.gz or the
  353 DHB matrix-cluster formulas) must be converted into the fixed-size RGB input
  expected by a pretrained ResNet-50 CNN for off-sample/on-sample classification.
when_to_use_negative:
- The task is pixel-level segmentation of off-sample regions — the article states
  the approach 'does not solve the segmentation problem of finding pixels which are
  in the off-sample region'
- The input is not ion image data (e.g. feature tables, spectra vectors) or images
  are already in the 224x224 RGB normalized format and can go straight to inference
- A fresh deep learning architecture is being trained from scratch rather than reusing
  the ImageNet-pretrained ResNet-50 transfer-learning setup
tools:
- name: fastai
  role: Library used for the CNN training/inference pipeline and its default data-augmentation
    settings (crop/pad, flips, warping, rotation, zooming, brightness/contrast); v1.0.34
  repo: https://github.com/metaspace2020/offsample
- name: PyTorch
  role: Deep learning framework underlying the fastai preprocessing and model execution
    (v1.0)
  repo: https://github.com/metaspace2020/offsample
- name: torchvision
  role: Provides the ResNet-50 model (v0.2.1) pretrained on ImageNet that the preprocessed
    224x224 RGB images are fed into
  repo: https://github.com/metaspace2020/offsample
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:3792bb28024dda52437e842a4337d847b5a8997d37245829a38ceaf89a3ccdb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/offsample
  tools:
  - Fastai
  - METASPACE
  - PyTorch
  - fastai
  - torchvision
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Ion image preprocessing for CNN inference

## Summary

Transforms raw mass spectrometry ion images into the RGB, ImageNet-normalized, 224x224 format required by the fastai-trained ResNet-50 off-sample classifier from OffsampleAI (F1-score 0.97), so pretrained torchvision weights can be reused on imaging MS data.

## When to use

Use when ion images (e.g. from METASPACE gold standard GS.tar.gz or the 353 DHB matrix-cluster formulas) must be converted into the fixed-size RGB input expected by a pretrained ResNet-50 CNN for off-sample/on-sample classification.

## When NOT to use

- The task is pixel-level segmentation of off-sample regions — the article states the approach 'does not solve the segmentation problem of finding pixels which are in the off-sample region'
- The input is not ion image data (e.g. feature tables, spectra vectors) or images are already in the 224x224 RGB normalized format and can go straight to inference
- A fresh deep learning architecture is being trained from scratch rather than reusing the ImageNet-pretrained ResNet-50 transfer-learning setup

## Inputs

- Gold standard ion images archive (GS.tar.gz, release v0.2 of metaspace2020/offsample)
- Individual off-sample/on-sample ion images from METASPACE public datasets (e.g. DHB matrix-cluster molecular formulas)
- Predicted-probability reference archive (gs_predictions.tar.gz) for comparison

## Outputs

- 224x224 RGB ion-image tensors normalized with ImageNet per-channel mean/std
- Batched inference predictions: off-sample/on-sample class and predicted probability per ion image
- Per-dataset aggregated off-sample frequency counts per molecular formula

## How to apply

Download and unpack the gold standard ion images (GS.tar.gz from metaspace2020/offsample release v0.2) and iterate over per-dataset ion images. Apply the fastai/PyTorch preprocessing pipeline: zero-pad each ion image to square, convert to RGB, normalize with the ImageNet per-channel mean and std statistics, then resize and crop to 224x224 pixels. Feed the preprocessed batches into the ResNet-50 model (fastai v1.0.34, PyTorch v1.0, torchvision v0.2.1) pretrained on ImageNet to obtain off-sample/on-sample class predictions and probabilities. The rationale is that ion images are single-channel spatial maps, so zero padding plus RGB conversion lets an ImageNet-pretrained backbone transfer to imaging MS; during training (not inference) the fast.ai default augmentations (crop/pad, flips, warping, rotation, zooming, brightness/contrast) are additionally applied.

## Related tools

- **fastai** (Library used for the CNN training/inference pipeline and its default data-augmentation settings (crop/pad, flips, warping, rotation, zooming, brightness/contrast); v1.0.34) — https://github.com/metaspace2020/offsample
- **PyTorch** (Deep learning framework underlying the fastai preprocessing and model execution (v1.0)) — https://github.com/metaspace2020/offsample
- **torchvision** (Provides the ResNet-50 model (v0.2.1) pretrained on ImageNet that the preprocessed 224x224 RGB images are fed into) — https://github.com/metaspace2020/offsample

## Examples

```
from fastai.vision import cnn_learner, models; learn = cnn_learner(data, models.resnet50, pretrained=True)
```

## Evaluation signals

- All output images are exactly 224x224 pixels in RGB with per-channel statistics equal to the ImageNet normalization values
- Downstream ResNet-50 inference reaches the published off-sample recognition F1-score of 0.97 on the gold standard
- Reproduced predictions match gs_predictions.tar.gz (images grouped by predicted class with predicted probabilities)
- Off-sample frequency counts per molecular formula across the 24 DESI datasets reproduce Supplementary Data D5 (C18H30O3S off-sample in 14/24, C17H28O3S in 13/24)
- Single-channel ion images are correctly zero-padded to square before RGB conversion, avoiding aspect-ratio distortion

## Limitations

- The pipeline classifies whole ion images, not pixels: 'our approach does not solve the segmentation problem of finding pixels which are in the off-sample region'
- Deep learning training requires 'a substantial training time, namely days on specially-devoted GPU'-equipped hardware, which hinders optimization (inference-only reuse is much cheaper)
- Gold standard covers only 23,238 images from 87 METASPACE datasets; transfer quality may degrade for matrices or tissue types far outside this distribution

## Evidence

- [discussion] ImageNet normalization and fixed-size geometry: "the images were normalized with the ImageNet statistics (per channel mean and std), resized and cropped to 224 × 224"
- [discussion] Library stack for preprocessing and model: "We used the fastai library (v1.0.34) with the PyTorch framework (v1.0)"
- [discussion] Augmentation strategy during training: "we applied data augmentation with the settings default for the fast.ai library"
- [readme] Pretrained backbone choice: "We trained Convolutional Neural Networks using Fastai and PyTorch libraries. The best performance we achieved using Resnet50 CNN pretrained on Imagenet."
- [discussion] ResNet-50 source: "with the ResNet-50 model from the torchvision library (v0.2.1)"
