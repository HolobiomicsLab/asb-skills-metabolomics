---
name: dhb-matrix-cluster-formula-annotation
description: Use when a MALDI-imaging dataset was acquired with 2,5-dihydroxybenzoic
  acid (DHB) matrix in positive ion mode and you need to estimate how many annotated
  DHB matrix-cluster ions are off-sample artifacts that should be filtered before
  downstream spatial metabolomics analysis.
when_to_use_negative:
- Datasets were not acquired with the DHB matrix in positive ion mode — the 353 formulas
  are DHB-specific (per Keller and Li, 2000) and do not model other MALDI matrices
- The goal is pixel-level segmentation of off-sample regions — the authors state 'our
  approach does not solve the segmentation problem of finding pixels which are in
  the off-sample region'; this skill classifies whole ion images only
- The task is generic off-sample recognition rather than characterizing DHB matrix-cluster
  contamination — the original skill from the article recognizes off-sample images
  broadly, not just matrix clusters
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: METASPACE
  role: Re-annotate the DHB positive-mode datasets using the DHB matrix-cluster formulas
    against HMDB v4 at FDR <= 50%
  repo: https://github.com/metaspace2020/metaspace/
- name: offsample microservice
  role: REST microservice serving the trained ResNet50 CNN to classify each annotated
    ion image as off-sample/on-sample with predicted probabilities
  repo: https://github.com/metaspace2020/metaspace/tree/master/metaspace/off-sample
- name: fastai
  role: Library (v1.0.34) with PyTorch framework used to train the ResNet50 off-sample
    classifier applied to the DHB cluster ion images
- name: torchvision
  role: Provides the ResNet-50 model (v0.2.1) pretrained on ImageNet used for off-sample
    recognition
- name: PyTorch
  role: Framework (v1.0) underlying the deep learning off-sample classifier
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:d8c4d80feb942e130469d458c323f63228d0e4ce3b6426e35c770bff55d0bf7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - Fastai
  - METASPACE
  - PyTorch
  - fastai
  - offsample microservice
  - torchvision
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# dhb-matrix-cluster-formula-annotation

## Summary

Re-annotates MALDI-imaging datasets acquired with the DHB matrix in positive ion mode using the 353 generated DHB matrix-cluster molecular formulas against HMDB v4 in METASPACE at FDR <= 50%, then classifies the resulting ion images as off-sample/on-sample with the ResNet50 off-sample CNN to quantify how much annotated DHB cluster signal is biologically unrelated (off-sample).

## When to use

Use when a MALDI-imaging dataset was acquired with 2,5-dihydroxybenzoic acid (DHB) matrix in positive ion mode and you need to estimate how many annotated DHB matrix-cluster ions are off-sample artifacts that should be filtered before downstream spatial metabolomics analysis.

## When NOT to use

- Datasets were not acquired with the DHB matrix in positive ion mode — the 353 formulas are DHB-specific (per Keller and Li, 2000) and do not model other MALDI matrices
- The goal is pixel-level segmentation of off-sample regions — the authors state 'our approach does not solve the segmentation problem of finding pixels which are in the off-sample region'; this skill classifies whole ion images only
- The task is generic off-sample recognition rather than characterizing DHB matrix-cluster contamination — the original skill from the article recognizes off-sample images broadly, not just matrix clusters

## Inputs

- 353 DHB matrix-cluster molecular formulas ('DHB mc mol formulas, predicted.csv' from metaspace2020/offsample)
- 31 DHB-positive-mode gold standard MALDI-imaging datasets from the METASPACE knowledge base
- Gold standard ion images archive (GS.tar.gz) with ion images to classify

## Outputs

- METASPACE annotations of DHB cluster formulas against HMDB v4 at FDR <= 50% (6677 annotations expected)
- Per-image off-sample/on-sample class labels with predicted probabilities from the ResNet50 microservice
- Summary table dhb_reannotation_summary.csv (headered CSV) with counts: 6677 annotated, 3134 (~47%) off-sample, ~5% of 58,203 total annotations, 4.3% isomeric with non-matrix HMDB molecules

## How to apply

First load the 353 DHB matrix-cluster molecular formulas generated per Keller and Li (2000) from the 'DHB mc mol formulas, predicted.csv' file in the metaspace2020/offsample repository. Re-annotate the DHB positive-mode datasets in METASPACE by searching these formulas against the HMDB v4 database at the False Positive Rate <= 50% threshold (the most permissive METASPACE value), which maximizes capture of matrix-cluster annotations. Download and unpack the gold standard ion images archive (GS.tar.gz) and run the best-performing ResNet50 CNN (pretrained on ImageNet, trained with fastai 1.0.34/PyTorch 1.0, served via the off-sample REST microservice) on every annotated ion image to obtain off-sample/on-sample classes and predicted probabilities. Finally compute summary statistics — total DHB cluster annotations, the off-sample fraction, the off-sample share of all annotations, and the fraction of recognized clusters isomeric with non-matrix HMDB molecules — and export them as a headered CSV (dhb_reannotation_summary.csv).

## Related tools

- **METASPACE** (Re-annotate the DHB positive-mode datasets using the DHB matrix-cluster formulas against HMDB v4 at FDR <= 50%) — https://github.com/metaspace2020/metaspace/
- **offsample microservice** (REST microservice serving the trained ResNet50 CNN to classify each annotated ion image as off-sample/on-sample with predicted probabilities) — https://github.com/metaspace2020/metaspace/tree/master/metaspace/off-sample
- **fastai** (Library (v1.0.34) with PyTorch framework used to train the ResNet50 off-sample classifier applied to the DHB cluster ion images)
- **torchvision** (Provides the ResNet-50 model (v0.2.1) pretrained on ImageNet used for off-sample recognition)
- **PyTorch** (Framework (v1.0) underlying the deep learning off-sample classifier)

## Examples

```
wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz
```

## Evaluation signals

- dhb_reannotation_summary.csv reports 6677 DHB matrix-cluster ions annotated at FDR <= 50% across the 31 DHB-positive-mode datasets
- 3134 of the 6677 annotations (~47%) are recognized as off-sample by the ResNet50 classifier
- Off-sample DHB cluster ions represent ~5% of all 58,203 annotated ions
- 4.3% of the recognized off-sample cluster ions are isomeric with non-matrix HMDB v4 molecules
- Every annotation falls within the FDR <= 50% acceptance threshold and every ion image carries a classifier label with a predicted probability

## Limitations

- The approach classifies whole ion images and does not solve the segmentation problem of finding off-sample pixels
- The FDR <= 50% threshold is the most permissive METASPACE value, so many annotations are low-confidence and some recognized clusters are molecularly ambiguous (4.3% isomeric with non-matrix HMDB molecules)
- The 353 formulas are derived from the Keller and Li (2000) cluster-generating model for DHB only; matrix clusters for other MALDI matrices are not covered
- Deep learning training requires substantial GPU resources ('days on specially-devoted GPU... power station'), though this skill only applies the pretrained model

## Evidence

- [readme] DHB cluster formula generation per Keller and Li (2000): [evidence span withheld — anchor status: not_found]
- [discussion] FDR <= 50% annotation threshold: "we considered annotations at a False Positive Rate < = 50% (the most permissive value in METASPACE)"
- [intro] Study motivation for DHB matrix cluster characterization: "we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB) [18] and derived properties of matrix clusters"
- [discussion] Deep learning toolchain: "We used the fastai library (v1.0.34) with the PyTorch framework (v1.0)"
- [discussion] ResNet50 pretrained on ImageNet: "We used the ResNet-50 network pre-trained on the ImageNet images [16]."
- [readme] GS.tar.gz download command: "wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz"
