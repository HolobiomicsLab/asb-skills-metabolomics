---
name: imaging-ms-annotation-count-summarization
description: Use when reproducing or auditing reported counts of metabolite annotations
  in METASPACE imaging-MS datasets — e.g. how many DHB matrix-cluster ions are annotated
  at a given FDR and how many are classified off-sample.
when_to_use_negative:
- Input is raw MALDI imaging mass spectrometry data that has not been annotated in
  METASPACE — run the annotation engine first
- Goal is pixel-level segmentation of off-sample regions rather than whole-image off-sample
  classification and count summarization
- Datasets were not acquired with the DHB matrix in positive ion mode — the 353 cluster
  formulas and reported counts are DHB-specific
edam_operation: http://edamontology.org/operation_3695
tools:
- name: METASPACE
  role: Re-annotate the 31 DHB positive-mode datasets against HMDB v4 at the FDR <=
    50% threshold to obtain the DHB cluster ion annotations
  repo: https://github.com/metaspace2020/metaspace/
- name: Fastai
  role: Library used with PyTorch to train the ResNet50 off-sample CNN applied to
    the annotated ion images
  repo: https://github.com/metaspace2020/offsample
- name: PyTorch
  role: Deep learning framework backing the ResNet50 off-sample classifier used to
    label each ion image
  repo: https://github.com/metaspace2020/offsample
- name: torchvision
  role: Source of the ResNet-50 model pretrained on ImageNet used by the off-sample
    classifier
- name: off-sample REST microservice
  role: Serves the trained ResNet50 model to classify every annotated DHB cluster
    ion image as off-sample/on-sample with predicted probabilities
  repo: https://github.com/metaspace2020/metaspace/tree/master/metaspace/off-sample
- name: sqlite3
  role: Exports tagger classifications from the TagOff SQLite database to CSV for
    count summarization
  repo: https://github.com/metaspace2020/offsample
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:d8c9b3b216a12b70555ecb586c12450be1435c4b43d79045122f6aecc61b6445
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - Fastai
  - METASPACE
  - PyTorch
  - off-sample REST microservice
  - sqlite3
  - torchvision
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# imaging-ms-annotation-count-summarization

## Summary

Reproduces the paper's DHB matrix-cluster off-sample recognition statistics by re-annotating 31 MALDI positive-mode datasets in METASPACE against HMDB v4 at FDR <= 50%, classifying ion images with the ResNet50 off-sample model, and exporting summary counts (6677 annotations; 3134 ~47% off-sample; ~5% of 58,203; 4.3% isomeric) to a CSV table.

## When to use

Use when reproducing or auditing reported counts of metabolite annotations in METASPACE imaging-MS datasets — e.g. how many DHB matrix-cluster ions are annotated at a given FDR and how many are classified off-sample.

## When NOT to use

- Input is raw MALDI imaging mass spectrometry data that has not been annotated in METASPACE — run the annotation engine first
- Goal is pixel-level segmentation of off-sample regions rather than whole-image off-sample classification and count summarization
- Datasets were not acquired with the DHB matrix in positive ion mode — the 353 cluster formulas and reported counts are DHB-specific

## Inputs

- DHB matrix cluster molecular formulas CSV (353 formulas: DHB mc mol formulas, predicted.csv)
- 31 MALDI positive-mode DHB gold standard datasets from the METASPACE knowledge base
- Gold standard ion images archive (GS.tar.gz)
- METASPACE annotations exported at FDR <= 50% against HMDB v4

## Outputs

- dhb_reannotation_summary.csv — headered CSV with annotation counts and fractions
- Per-ion-image classifier labels with predicted off-sample/on-sample probabilities

## How to apply

Load the 353 DHB matrix-cluster molecular formulas generated from the Keller and Li (2000) cluster model (DHB mc mol formulas, predicted.csv) and re-annotate the 31 DHB positive-mode gold standard datasets in METASPACE against HMDB v4 at the False Positive Rate <= 50% threshold, the most permissive METASPACE value. Download and unpack the gold standard ion image archive (GS.tar.gz from the metaspace2020/offsample releases), then run the best-performing ResNet50 CNN (pretrained on ImageNet, trained with fastai 1.0.34/PyTorch 1.0 and served via the off-sample REST microservice) on every annotated ion image to obtain off-sample/on-sample labels with predicted probabilities. Compute the summary statistics — total DHB cluster annotations (6677), fraction recognized off-sample (3134, ~47%), share of all 58,203 annotations (~5%), and fraction isomeric with non-matrix HMDB molecules (4.3%). Export the counts as a headered CSV table (dhb_reannotation_summary.csv) for validation against the reported targets.

## Related tools

- **METASPACE** (Re-annotate the 31 DHB positive-mode datasets against HMDB v4 at the FDR <= 50% threshold to obtain the DHB cluster ion annotations) — https://github.com/metaspace2020/metaspace/
- **Fastai** (Library used with PyTorch to train the ResNet50 off-sample CNN applied to the annotated ion images) — https://github.com/metaspace2020/offsample
- **PyTorch** (Deep learning framework backing the ResNet50 off-sample classifier used to label each ion image) — https://github.com/metaspace2020/offsample
- **torchvision** (Source of the ResNet-50 model pretrained on ImageNet used by the off-sample classifier)
- **off-sample REST microservice** (Serves the trained ResNet50 model to classify every annotated DHB cluster ion image as off-sample/on-sample with predicted probabilities) — https://github.com/metaspace2020/metaspace/tree/master/metaspace/off-sample
- **sqlite3** (Exports tagger classifications from the TagOff SQLite database to CSV for count summarization) — https://github.com/metaspace2020/offsample

## Examples

```
sqlite3 -header -csv ./metaspace/webapp/imageclassification.sqlite "select * from imageclassifications" > ./metaspace/webapp/dist/results.csv
```

## Evaluation signals

- dhb_reannotation_summary.csv reports exactly 6677 DHB matrix cluster ion annotations at FDR <= 50% against HMDB v4
- Off-sample fraction matches 3134 of 6677 (~47%) recognized as off-sample
- Off-sample cluster share equals ~5% of all 58,203 annotations
- Fraction of recognized cluster ions isomeric with non-matrix HMDB v4 molecules equals 4.3%
- Every counted annotation falls within the FDR <= 50% acceptance threshold and every image carries a classifier label with a predicted probability

## Limitations

- The classifier works at the whole-image level and does not solve the segmentation problem of finding off-sample pixels
- Counts are specific to the DHB matrix, positive ion mode, HMDB v4, and the most permissive METASPACE FDR <= 50% threshold
- A nontrivial share (4.3%) of recognized cluster ions are isomeric with non-matrix HMDB molecules, so formula-level matches may confuse matrix and biological ions

## Evidence

- [discussion] Permissive FDR threshold used for annotation filtering: "we considered annotations at a False Positive Rate < = 50% (the most permissive value in METASPACE)"
- [readme] 353 DHB cluster formulas generated from Keller and Li model: "We have generated DHB matrix clusters according to [(Keller and Li, 2000)](./DHB%20matrix%20clusters/Keller%20and%20Li%2C%202000.pdf). This resulted in 353 molecular formulas"
- [intro] DHB matrix cluster investigation motivation: "we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB)"
- [discussion] Classifier architecture used for off-sample recognition: "We used the fastai library (v1.0.34) with the PyTorch framework (v1.0)"
