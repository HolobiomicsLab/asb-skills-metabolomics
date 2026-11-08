---
name: isomeric-molecule-overlap-analysis
description: Use when MALDI-imaging matrix clusters (e.g., 353 predicted DHB formulas)
  must be re-annotated against HMDB v4 at FDR <= 50% and checked for exact-formula
  (isomeric) overlap with non-matrix metabolites.
when_to_use_negative:
- Annotations are resolved by structural or MS/MS-level search rather than formula
  search — isomeric formula collisions are then already separated by the method and
  this skill adds nothing.
- The MALDI matrix is not DHB — the 353-formula Keller and Li (2000) cluster model
  covers only 2,5-dihydroxybenzoic acid cluster chemistry.
- Ion images lack off-sample/on-sample classifier labels — the isomeric fraction is
  defined only over recognized off-sample cluster ions.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3382
tools:
- name: METASPACE
  role: Re-annotates the 31 DHB-positive-mode datasets with the 353 DHB matrix-cluster
    formulas against HMDB v4 at FDR <= 50% to produce the annotation set
  repo: https://github.com/metaspace2020/metaspace/
- name: fastai
  role: Training framework for the ResNet50 off-sample CNN whose labels define which
    DHB cluster ions count as recognized off-sample before the isomeric overlap is
    computed
- name: PyTorch
  role: Deep learning framework (v1.0) underpinning the ResNet50 off-sample classifier
    applied to DHB cluster ion images
- name: torchvision
  role: Provides the ResNet-50 model (v0.2.1) pretrained on ImageNet that is used
    for off-sample recognition of the DHB cluster annotations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:f1e629bce326b8408697eef9391e9014e77891d257973d5016a71116c0f44e9c
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# isomeric-molecule-overlap-analysis

## Summary

Quantifies how many recognized off-sample DHB matrix-cluster ion annotations share an exact molecular formula with non-matrix HMDB v4 molecules, exposing the 4.3% of matrix signal that formula-level annotation cannot disambiguate from biology.

## When to use

Use when MALDI-imaging matrix clusters (e.g., 353 predicted DHB formulas) must be re-annotated against HMDB v4 at FDR <= 50% and checked for exact-formula (isomeric) overlap with non-matrix metabolites.

## When NOT to use

- Annotations are resolved by structural or MS/MS-level search rather than formula search — isomeric formula collisions are then already separated by the method and this skill adds nothing.
- The MALDI matrix is not DHB — the 353-formula Keller and Li (2000) cluster model covers only 2,5-dihydroxybenzoic acid cluster chemistry.
- Ion images lack off-sample/on-sample classifier labels — the isomeric fraction is defined only over recognized off-sample cluster ions.

## Inputs

- DHB matrix-cluster molecular formula list: 'DHB mc mol formulas, predicted.csv' (353 formulas generated per Keller and Li, 2000)
- 31 DHB-positive-mode gold standard MALDI-imaging datasets from the METASPACE knowledge base
- HMDB v4 metabolite database (formula search; non-matrix entries for the isomeric comparison)
- Gold standard ion image archive (GS.tar.gz) with per-image off-sample/on-sample class and predicted probability from the ResNet50 microservice

## Outputs

- dhb_reannotation_summary.csv: headered CSV with total DHB cluster annotations (6677), recognized off-sample count (3134, ~47%), off-sample share of all 58,203 annotations (~5%), and isomeric fraction (4.3%)
- Per-annotation flags: FDR <= 50% acceptance status, off-sample classifier label with predicted probability, and isomeric-with-non-matrix-HMDB-v4 indicator

## How to apply

Re-annotate the 31 DHB-positive-mode gold standard datasets in METASPACE using the 353 predicted DHB matrix-cluster molecular formulas (from the combinatorial Keller and Li, 2000 model, 'DHB mc mol formulas, predicted.csv') searched against HMDB v4 at False Positive Rate <= 50%, the most permissive METASPACE threshold. Run the ImageNet-pretrained ResNet50 off-sample classifier (fastai 1.0.34 / PyTorch 1.0, ResNet-50 from torchvision 0.2.1, served via the off-sample REST microservice) on each annotated ion image from GS.tar.gz to assign off-sample/on-sample labels with predicted probabilities. Restrict the isomeric comparison to recognized off-sample cluster ions and match their molecular formulas against formulas of all non-matrix HMDB v4 entries: an exact formula match means the cluster is isomeric with a biological molecule and cannot be separated by formula-based annotation. Compute the summary targets: 6677 annotated DHB cluster ions, 3134 (~47%) recognized off-sample, ~5% of all 58,203 annotations, and 4.3% isomeric with non-matrix HMDB molecules, exporting to a headered dhb_reannotation_summary.csv. Rationale: the isomeric fraction quantifies the ambiguity floor of matrix interference — these ions inflate biological annotations even after off-sample recognition.

## Related tools

- **METASPACE** (Re-annotates the 31 DHB-positive-mode datasets with the 353 DHB matrix-cluster formulas against HMDB v4 at FDR <= 50% to produce the annotation set) — https://github.com/metaspace2020/metaspace/
- **fastai** (Training framework for the ResNet50 off-sample CNN whose labels define which DHB cluster ions count as recognized off-sample before the isomeric overlap is computed)
- **PyTorch** (Deep learning framework (v1.0) underpinning the ResNet50 off-sample classifier applied to DHB cluster ion images)
- **torchvision** (Provides the ResNet-50 model (v0.2.1) pretrained on ImageNet that is used for off-sample recognition of the DHB cluster annotations)

## Examples

```
wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz
```

## Evaluation signals

- Total annotated DHB matrix cluster ions at FDR <= 50% equals 6677 across the 31 DHB-positive-mode datasets.
- Recognized off-sample cluster ions equal 3134 (~47% of 6677) and represent ~5% of all 58,203 annotations.
- The isomeric fraction of recognized cluster ions against non-matrix HMDB v4 formulas equals 4.3%.
- Every counted annotation satisfies the FDR <= 50% acceptance threshold and every image carries a classifier label with predicted probability.
- dhb_reannotation_summary.csv is a headered CSV whose values match the reported targets exactly.

## Limitations

- Formula-based matching cannot separate isomeric species: 4.3% of recognized clusters are indistinguishable by formula from biological HMDB v4 molecules.
- The off-sample classifier works at image level and does not solve pixel-level segmentation of off-sample regions.
- FDR <= 50% is the most permissive METASPACE setting, so the 6677 annotations carry a high false-positive load.
- Coverage is limited to DHB matrix clusters generated by the Keller and Li (2000) combinatorial model (353 formulas).

## Evidence

- [abstract] DHB matrix cluster properties and off-sample relevance: "we investigated off-sample images corresponding to the most common MALDI matrix (2,5-dihydroxybenzoic acid, DHB) and characterized properties of matrix clusters"
- [intro] Derivation of DHB matrix cluster properties via cluster-generating model: "we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB) [18] and derived properties of matrix clusters using a cluster-generating"
- [discussion] Annotation acceptance threshold used for counting cluster ions: "we considered annotations at a False Positive Rate < = 50% (the most permissive value in METASPACE)"
- [discussion] Classifier architecture applied to the annotated ion images: "with the ResNet-50 model from the torchvision library (v0.2.1)"
- [readme] Source of the 353 DHB matrix-cluster formulas input: "This resulted in 353 molecular formulas available [here](./DHB%20matrix%20clusters/DHB%20mc%20mol%20formulas%2C%20predicted.csv)."
