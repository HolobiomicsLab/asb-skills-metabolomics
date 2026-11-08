---
name: per-formula-off-sample-frequency-analysis
description: Use when you have per-ion-image off-sample predictions across multiple
  datasets and want to know which molecular formulae are classified off-sample in
  a majority of datasets, e.g., to flag matrix-cluster contamination like DHB cluster
  ions.
when_to_use_negative:
- Input images are already labeled off-sample/on-sample by experts and only agreement
  statistics are needed — use inter-tagger agreement metrics (Cohen's kappa, Krippendorff's
  alpha) instead.
- The goal is pixel-level localization of off-sample regions — the article explicitly
  states the approach 'does not solve the segmentation problem of finding pixels which
  are in the off-sample region'.
- The matrix of interest is not DHB and no equivalent formula list exists — the per-formula
  frequency aggregation here is anchored to the 353 DHB matrix-cluster formulas.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: METASPACE
  role: Source of the public imaging MS datasets (including the DESI datasets) over
    which per-formula off-sample frequencies are aggregated
  repo: https://github.com/metaspace2020/metaspace/
- name: fastai
  role: Library (v1.0.34) used to train the off-sample CNN classifier whose predictions
    are aggregated per formula
  repo: https://github.com/metaspace2020/offsample
- name: PyTorch
  role: Deep learning framework (v1.0) underlying the ResNet-50 off-sample classifier
    used for batch inference on ion images
  repo: https://github.com/metaspace2020/offsample
- name: ResNet-50
  role: ImageNet-pretrained CNN architecture (torchvision v0.2.1) that produces the
    off-sample/on-sample class and probability per ion image
  repo: https://github.com/metaspace2020/offsample
- name: TagOff
  role: Web app used to expert-tag the gold standard ion images that the classifier
    was trained and evaluated on
  repo: https://github.com/metaspace2020/offsample
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:e964bac91dcb6660aea7dd8d83a57f4b86c19ceda8fb502a98b174e8984b2b8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - Fastai
  - METASPACE
  - PyTorch
  - ResNet-50
  - TagOff
  - fastai
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Per-formula off-sample frequency analysis

## Summary

Aggregates off-sample/on-sample classifier predictions (ResNet-50 CNN via fastai/PyTorch) per molecular formula across multiple imaging MS datasets, then computes how many datasets each formula was classified off-sample in. Used to identify systematically polluted ions (e.g., MALDI matrix clusters such as DHB-derived C18H30O3S and C17H28O3S in DESI data).

## When to use

Use when you have per-ion-image off-sample predictions across multiple datasets and want to know which molecular formulae are classified off-sample in a majority of datasets, e.g., to flag matrix-cluster contamination like DHB cluster ions.

## When NOT to use

- Input images are already labeled off-sample/on-sample by experts and only agreement statistics are needed — use inter-tagger agreement metrics (Cohen's kappa, Krippendorff's alpha) instead.
- The goal is pixel-level localization of off-sample regions — the article explicitly states the approach 'does not solve the segmentation problem of finding pixels which are in the off-sample region'.
- The matrix of interest is not DHB and no equivalent formula list exists — the per-formula frequency aggregation here is anchored to the 353 DHB matrix-cluster formulas.

## Inputs

- Gold standard ion images archive (GS.tar.gz) from github.com/metaspace2020/offsample release v0.2
- Trained ResNet-50 off-sample classifier (fastai/PyTorch model)
- Predicted probabilities per ion image (gs_predictions.tar.gz as reference)
- List of target molecular formulas (e.g., the 353 DHB matrix-cluster formulas from 'DHB mc mol formulas, predicted.csv')
- Dataset group over which to aggregate frequencies (e.g., 24 DESI datasets from METASPACE)

## Outputs

- Per-formula off-sample frequency counts across datasets (e.g., C18H30O3S: 14/24; C17H28O3S: 13/24)
- Per-ion-image predicted class (off-sample/on-sample) and predicted probability
- Ranked list of molecular formulae classified off-sample in more than half of the datasets

## How to apply

Download and unpack the gold standard ion images archive (GS.tar.gz from the metaspace2020/offsample v0.2 release) and run batch inference over each ion image with the best off-sample classifier: a ResNet-50 pretrained on ImageNet, trained with fastai (v1.0.34) on PyTorch (v1.0). Preprocess each ion image for the CNN by converting to RGB, normalizing with the ImageNet per-channel mean/std statistics, and resizing/cropping to 224x224; obtain a predicted off-sample/on-sample class plus predicted probability per image. Restrict aggregation to a defined formula list — for DHB matrix-cluster analysis, the 353 predicted DHB cluster molecular formulas shipped with the repository. For each molecular formula, count in how many datasets it is predicted off-sample (e.g., across the 24 DESI datasets from the Takats lab at Imperial College London and the Eberlin lab at UT Austin); formulas off-sample in more than half the datasets are the systematically off-sample ones. Validate the counts against the paper's Supplementary Data D5 (C18H30O3S off-sample in 14/24 datasets, C17H28O3S in 13/24).

## Related tools

- **METASPACE** (Source of the public imaging MS datasets (including the DESI datasets) over which per-formula off-sample frequencies are aggregated) — https://github.com/metaspace2020/metaspace/
- **fastai** (Library (v1.0.34) used to train the off-sample CNN classifier whose predictions are aggregated per formula) — https://github.com/metaspace2020/offsample
- **PyTorch** (Deep learning framework (v1.0) underlying the ResNet-50 off-sample classifier used for batch inference on ion images) — https://github.com/metaspace2020/offsample
- **ResNet-50** (ImageNet-pretrained CNN architecture (torchvision v0.2.1) that produces the off-sample/on-sample class and probability per ion image) — https://github.com/metaspace2020/offsample
- **TagOff** (Web app used to expert-tag the gold standard ion images that the classifier was trained and evaluated on) — https://github.com/metaspace2020/offsample

## Examples

```
wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/gs_predictions.tar.gz | tar -xvz
```

## Evaluation signals

- Reproduced frequency counts match Supplementary Data D5: C18H30O3S off-sample in 14/24 DESI datasets and C17H28O3S in 13/24 datasets.
- Predicted probabilities are present for every ion image and the classification model attains F1-score ~0.97 on the gold standard (23,238 expert-tagged images from 87 public METASPACE datasets).
- The formula list contains exactly 353 DHB matrix-cluster molecular formulas as provided in 'DHB mc mol formulas, predicted.csv'.
- Aggregation is restricted to the intended dataset group (24 DESI datasets); counts per formula never exceed the number of datasets.

## Limitations

- The approach operates at the ion-image level and does not solve the segmentation problem of finding pixels which are in the off-sample region.
- Ground truth information about the molecular content of analyzed samples is often lacking, so 'off-sample in a majority of datasets' is a heuristic contamination signal, not proof of matrix origin.
- Deep learning networks require substantial training time (days on a GPU-equipped workstation), which hinders optimization of the classifier whose predictions feed this analysis.

## Evidence

- [readme] Download and unpack the gold standard ion images archive: "wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz"
- [discussion] Classifier architecture and libraries: "We used the fastai library (v1.0.34) with the PyTorch framework (v1.0)"
- [discussion] Image preprocessing for the CNN: "the images were normalized with the ImageNet statistics (per channel mean and std), resized and cropped to 224 × 224"
- [readme] DHB matrix-cluster formula list used for aggregation: "This resulted in 353 molecular formulas available [here](./DHB%20matrix%20clusters/DHB%20mc%20mol%20formulas%2C%20predicted.csv)."
- [abstract] Gold standard scale underlying the classifier: "we created a high-quality gold standard of 23,238 expert-tagged ion images from 87 public datasets from the METASPACE knowledge base"
