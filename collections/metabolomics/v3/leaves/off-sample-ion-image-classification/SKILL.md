---
name: off-sample-ion-image-classification
description: Use when ion images from imaging MS datasets (e.g., METASPACE annotations)
  must be labeled off-sample/on-sample, or when matrix-cluster signals such as DHB
  clusters need to be filtered before downstream spatial metabolomics analysis.
when_to_use_negative:
- The goal is pixel-level segmentation of the off-sample region — the article states
  the approach does not solve the segmentation problem of finding off-sample pixels
- Data is not imaging mass spectrometry ion images (e.g., already-aggregated feature
  tables or spectra without spatial images)
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: fastai
  role: Library (v1.0.34) used to train the CNN classifier on the expert-tagged gold
    standard with default data augmentation
  repo: https://github.com/metaspace2020/offsample
- name: PyTorch
  role: Deep learning framework (v1.0) underlying the fastai training of the off-sample
    classifier
  repo: https://github.com/metaspace2020/offsample
- name: torchvision
  role: Source of the ResNet-50 model (v0.2.1) pretrained on ImageNet used for off-sample
    recognition
- name: METASPACE
  role: Knowledge base supplying public datasets and ion images; hosts the trained
    classifier as an off-sample filter and REST microservice
  repo: https://github.com/metaspace2020/metaspace/
- name: TagOff
  role: Web app for expert tagging of ion images into off-sample/on-sample/unknown
    classes to build the gold standard
  repo: https://github.com/metaspace2020/offsample
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:2aa0a9e2e7d3ff0684740848aae1f2dc92ba14d8b898d5067a6b8718987eec33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/offsample
  tools:
  - Fastai
  - METASPACE
  - PyTorch
  - TagOff
  - fastai
  - torchvision
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# off-sample-ion-image-classification

## Summary

Classify ion images from imaging mass spectrometry as off-sample (e.g., MALDI matrix cluster signals) vs. on-sample using a ResNet-50 CNN pretrained on ImageNet and trained with fastai/PyTorch, reproducing the OffsampleAI workflow on METASPACE data.

## When to use

Use when ion images from imaging MS datasets (e.g., METASPACE annotations) must be labeled off-sample/on-sample, or when matrix-cluster signals such as DHB clusters need to be filtered before downstream spatial metabolomics analysis.

## When NOT to use

- The goal is pixel-level segmentation of the off-sample region — the article states the approach does not solve the segmentation problem of finding off-sample pixels
- Data is not imaging mass spectrometry ion images (e.g., already-aggregated feature tables or spectra without spatial images)

## Inputs

- Gold standard ion images archive (GS.tar.gz, v0.2 release of metaspace2020/offsample)
- METASPACE ion images for annotations at False Positive Rate <= 50% against HMDB v4
- 353 DHB matrix-cluster molecular formulas CSV (DHB mc mol formulas, predicted.csv)

## Outputs

- Per-image off-sample/on-sample class label with predicted probability
- Predictions archive grouped by predicted class (gs_predictions.tar.gz)
- Aggregated off-sample frequency counts per molecular formula per dataset (e.g., dhb_reannotation_summary.csv)

## How to apply

Download and unpack the gold standard ion images (GS.tar.gz from the metaspace2020/offsample v0.2 release) and load the ResNet-50 CNN pretrained on ImageNet, trained with the fastai library (v1.0.34) on the PyTorch framework (v1.0). Preprocess each ion image for the CNN: convert to RGB, zero-pad if needed, normalize with the ImageNet per-channel mean/std statistics, and resize/crop to 224x224, applying fast.ai-default data augmentation when training. Run batch inference to obtain an off-sample/on-sample class and a predicted probability per image; for reproducibility, aggregate predictions per molecular formula per dataset (e.g., the 353 DHB matrix-cluster formulas against HMDB v4 at FDR <= 50%, the most permissive METASPACE value) and compute off-sample frequency counts across datasets. Rationale: deep learning achieved the best F1 off-sample score (0.97) on the 23,238-image gold standard from 87 public METASPACE datasets, and the trained model is deployed as an off-sample filter and REST microservice in METASPACE.

## Related tools

- **fastai** (Library (v1.0.34) used to train the CNN classifier on the expert-tagged gold standard with default data augmentation) — https://github.com/metaspace2020/offsample
- **PyTorch** (Deep learning framework (v1.0) underlying the fastai training of the off-sample classifier) — https://github.com/metaspace2020/offsample
- **torchvision** (Source of the ResNet-50 model (v0.2.1) pretrained on ImageNet used for off-sample recognition)
- **METASPACE** (Knowledge base supplying public datasets and ion images; hosts the trained classifier as an off-sample filter and REST microservice) — https://github.com/metaspace2020/metaspace/
- **TagOff** (Web app for expert tagging of ion images into off-sample/on-sample/unknown classes to build the gold standard) — https://github.com/metaspace2020/offsample

## Examples

```
wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz
```

## Evaluation signals

- Reproduced off-sample F1-score should approach 0.97 on the 23,238-image gold standard (grouped five-fold cross-validation grouped by dataset)
- For DHB-positive-mode datasets: 6677 DHB cluster ions annotated at FDR <= 50%, of which 3134 (~47%) recognized as off-sample, ~5% of 58,203 total annotations, 4.3% isomeric with non-matrix HMDB molecules
- DESI off-sample check: C18H30O3S classified off-sample in 14/24 datasets and C17H28O3S in 13/24, matching Supplementary Data D5
- Every ion image must carry a classifier label with an associated predicted probability
- All annotations used must fall within the FDR <= 50% acceptance threshold

## Limitations

- The approach does not solve the segmentation problem of finding pixels in the off-sample region
- Deep learning networks require substantial training time (days on a dedicated GPU), which hinders optimization
- Ground truth about the molecular content of samples is lacking, so off-sample labels rely on expert judgement curated into the gold standard

## Evidence

- [discussion] ResNet-50 ImageNet preprocessing: "the images were normalized with the ImageNet statistics (per channel mean and std), resized and cropped to 224 × 224"
- [discussion] fastai/PyTorch training stack: "We used the fastai library (v1.0.34) with the PyTorch framework (v1.0)"
- [discussion] FDR threshold for tagging: "we considered annotations at a False Positive Rate < = 50% (the most permissive value in METASPACE)"
- [readme] Gold standard download: "wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz"
- [abstract] Gold standard size: "we created a high-quality gold standard of 23,238 expert-tagged ion images from 87 public datasets from the METASPACE knowledge base"
