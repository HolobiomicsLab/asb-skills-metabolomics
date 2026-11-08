---
name: off-sample-annotation-filtering
description: Use when MALDI-imaging annotations in METASPACE are suspected to be off-sample
  matrix cluster signals (e.g. DHB clusters) and you must filter them, or when you
  need to characterize matrix cluster composition from recognized off-sample annotations.
when_to_use_negative:
- Input annotations are already confirmed on-sample biological signals — filtering
  would remove relevant data
- The MALDI matrix is not DHB and no analogous cluster-generating model/formula list
  exists — the parameter histograms cannot be matched
- The goal is pixel-level segmentation of off-sample regions rather than whole-ion-image
  annotation-level filtering (the paper's approach 'does not solve the segmentation
  problem of finding pixels which are in the off-sample region')
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: METASPACE
  role: Source of public imaging MS datasets, pixel-annotation export, and the platform
    hosting the off-sample annotation filter and REST API
  repo: https://github.com/metaspace2020/metaspace/
- name: TagOff
  role: Web app used by expert taggers to label ion images as off-sample/on-sample/unknown,
    producing the judgments that train the off-sample recognizer
  repo: https://github.com/metaspace2020/offsample
- name: torchvision 0.2.1
  role: Provides the ResNet-50 model used for off-sample image recognition (F1 0.97)
    that flags the annotations being filtered
  repo: https://github.com/metaspace2020/offsample
- name: fastai 1.0.34
  role: Library (with PyTorch 1.0) used to train the ResNet-50 off-sample classifier
  repo: https://github.com/metaspace2020/offsample
- name: sqlite3
  role: Exports TagOff tagging results from imageclassification.sqlite to CSV for
    downstream use
  repo: https://github.com/metaspace2020/offsample
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:a192e3976c9f1acd81d3884790bb2b53a5fd2d0bf4dabc340c80988a12c5ed65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - TagOff
  - fastai 1.0.34
  - sqlite3
  - torchvision 0.2.1
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# off-sample-annotation-filtering

## Summary

Filters matrix-cluster contamination in MALDI imaging MS by recognizing off-sample ion images with a ResNet-50 deep learning classifier (F1 0.97) and, for the DHB matrix, characterizing the combinatorial cluster-generating parameters (n, p, x, z) of recognized off-sample cluster ions as in Fig. 3 of Ovchinnikova et al. 2020.

## When to use

Use when MALDI-imaging annotations in METASPACE are suspected to be off-sample matrix cluster signals (e.g. DHB clusters) and you must filter them, or when you need to characterize matrix cluster composition from recognized off-sample annotations.

## When NOT to use

- Input annotations are already confirmed on-sample biological signals — filtering would remove relevant data
- The MALDI matrix is not DHB and no analogous cluster-generating model/formula list exists — the parameter histograms cannot be matched
- The goal is pixel-level segmentation of off-sample regions rather than whole-ion-image annotation-level filtering (the paper's approach 'does not solve the segmentation problem of finding pixels which are in the off-sample region')

## Inputs

- pixel-annot-export-v0.10.tar.gz — METASPACE pixel-annotation export of recognized off-sample DHB cluster ion annotations from 31 datasets
- DHB mc mol formulas, predicted.csv — 353 predicted DHB matrix cluster molecular formulas per Keller and Li (2000)
- Recognized off-sample DHB cluster ion molecular formulas (matched to the predicted formula set)

## Outputs

- Occurrence count tally of combinatorial model parameters n, p, x, z across recognized off-sample DHB cluster ions
- Fig. 3-style histograms of the parameters n, p, x, and z with labeled axes
- Qualitative characterization: prevalence of n=2 (two-DHB-molecule) clusters and near-absence of potassium-containing clusters (x=0 majority)

## How to apply

Load recognized off-sample DHB cluster ion annotations from 31 METASPACE datasets using the pixel-annotation export (pixel-annot-export-v0.10.tar.gz), and load the 353 predicted DHB matrix cluster molecular formulas generated per the Keller and Li (2000) combinatorial model (DHB mc mol formulas CSV). Match each recognized off-sample cluster ion formula to its combinatorial model parameters: n (number of DHB molecules), p, x (potassium count), and z (sodium count), then tally occurrence counts of each parameter value across all recognized cluster ions. Plot histograms of n, p, x, and z in the Fig. 3 layout (e.g. matplotlib) with clearly labeled axes. Validate by checking that two-DHB-molecule clusters (n=2) are the most prevalent class and potassium-containing clusters (x>0) are nearly absent (y=0 for a prevalent majority). Off-sample recognition itself was implemented as a METASPACE filter ('Show/hide off-sample annotations') backed by a REST API microservice in the metaspace2020/metaspace repository.

## Related tools

- **METASPACE** (Source of public imaging MS datasets, pixel-annotation export, and the platform hosting the off-sample annotation filter and REST API) — https://github.com/metaspace2020/metaspace/
- **TagOff** (Web app used by expert taggers to label ion images as off-sample/on-sample/unknown, producing the judgments that train the off-sample recognizer) — https://github.com/metaspace2020/offsample
- **torchvision 0.2.1** (Provides the ResNet-50 model used for off-sample image recognition (F1 0.97) that flags the annotations being filtered) — https://github.com/metaspace2020/offsample
- **fastai 1.0.34** (Library (with PyTorch 1.0) used to train the ResNet-50 off-sample classifier) — https://github.com/metaspace2020/offsample
- **sqlite3** (Exports TagOff tagging results from imageclassification.sqlite to CSV for downstream use) — https://github.com/metaspace2020/offsample

## Examples

```
wget -qO - https://s3-eu-west-1.amazonaws.com/sm-off-sample/pixel-annot-export-v0.10.tar.gz | tar -xvz
```

## Evaluation signals

- Histogram of n shows clusters with two DHB molecules (n=2) as the most prevalent class, matching the paper's Fig. 3 qualitative pattern
- Potassium parameter x equals 0 for a prevalent majority of recognized cluster ions (potassium-containing clusters nearly absent)
- Every recognized off-sample DHB cluster ion formula matches exactly one row of the 353 predicted formulas in 'DHB mc mol formulas, predicted.csv' before parameter tallying
- The off-sample filter, when evaluated on the gold standard, achieves F1-score ≈ 0.97 as reported for residual deep learning
- Annotation export contains annotations from the 31 METASPACE datasets with DHB matrix at False Positive Rate <= 50% (the permissiveness used when selecting ion images for tagging)

## Limitations

- The approach classifies whole ion images and does not solve the segmentation problem of finding individual pixels in the off-sample region
- Only qualitative histogram patterns are validated (n=2 prevalence, x=0 majority); no quantitative thresholds are published for the distributions
- Deep learning training takes days on a GPU-equipped workstation, hindering optimization and retraining
- Ground truth about molecular content of analyzed samples is often lacking, limiting independent verification of cluster composition

## Evidence

- [results] Recognized DHB clusters show a slight prevalence of clusters including two DHB molecules, and for a prevalent majority of recognized cluster ions the potassium parameter y is equal to 0.: [evidence span withheld — anchor status: not_found]
- [readme] We generated DHB matrix clusters according to Keller and Li 2000, resulting in 353 molecular formulas available in the offsample repository.: "We have generated DHB matrix clusters according to [(Keller and Li, 2000)](./DHB%20matrix%20clusters/Keller%20and%20Li%2C%202000.pdf). This resulted in 353 molecular formulas available"
- [discussion] The off-sample classifier is accessible in METASPACE as a new filter via the 'Show/hide off-sample annotations' option.: [evidence span withheld — anchor status: not_found]
- [methods] The METASPACE pixel-annotation export archive is distributed at the sm-off-sample S3 bucket as pixel-annot-export-v0.10.tar.gz.: "https://s3-eu-west-1.amazonaws.com/sm-off-sample/pixel-annot-export-v0.10.tar.gz"
- [abstract] The paper investigated off-sample images of the most common MALDI matrix DHB and characterized properties of matrix clusters.: "we investigated off-sample images corresponding to the most common MALDI matrix (2,5-dihydroxybenzoic acid, DHB) and characterized properties of matrix clusters"
