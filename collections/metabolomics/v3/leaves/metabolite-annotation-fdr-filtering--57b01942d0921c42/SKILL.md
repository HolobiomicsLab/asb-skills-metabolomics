---
name: metabolite-annotation-fdr-filtering
description: Use when MALDI-imaging datasets must be re-annotated with a specific
  formula list (e.g., matrix clusters) against a metabolite database and a controlled
  FDR cutoff is needed before image classification.
when_to_use_negative:
- The input is raw centroided/profile imaging MS data not yet processed into METASPACE-ready
  imzML datasets — annotation requires the METASPACE engine pipeline first
- The analysis requires strict annotation confidence (e.g., FDR <= 5-10%); this skill
  deliberately uses the most permissive FDR <= 50% threshold to capture matrix clusters,
  which would inflate false positives in high-confidence workflows
- The matrix is not DHB (e.g., CHCA, 9-AA) — the 353 cluster formulas are specific
  to 2,5-dihydroxybenzoic acid generated per the Keller and Li (2000) model
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: METASPACE
  role: Cloud platform used to re-annotate the 31 DHB positive-mode datasets with
    DHB matrix-cluster formulas against HMDB v4 at FDR <= 50%, and source of the 58,203
    total annotated ions
  repo: https://github.com/metaspace2020/metaspace/
- name: Fastai
  role: Library (v1.0.34) with which the best-performing ResNet50 off-sample classifier
    was trained; used to serve classifications for the FDR-filtered annotations
  repo: https://github.com/metaspace2020/offsample
- name: PyTorch
  role: Framework (v1.0) underlying the fastai training and the off-sample REST microservice
    inference on annotated ion images
  repo: https://github.com/metaspace2020/offsample
- name: torchvision
  role: Source (v0.2.1) of the ResNet-50 architecture pretrained on ImageNet used
    for off-sample recognition of the filtered annotations
  repo: https://github.com/metaspace2020/offsample
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:2b6aa281596d0205b4ba6db42c97efb18283652a746d6c8d1df58d92bb315def
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - Fastai
  - METASPACE
  - PyTorch
  - torchvision
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# metabolite-annotation-fdr-filtering

## Summary

Re-annotate target ion sets (e.g., 353 generated DHB matrix-cluster molecular formulas) in METASPACE against the HMDB v4 database at the False Positive Rate <= 50% threshold (the most permissive METASPACE value) to obtain per-annotation FDR-filtered results for downstream off-sample classification.

## When to use

Use when MALDI-imaging datasets must be re-annotated with a specific formula list (e.g., matrix clusters) against a metabolite database and a controlled FDR cutoff is needed before image classification.

## When NOT to use

- The input is raw centroided/profile imaging MS data not yet processed into METASPACE-ready imzML datasets — annotation requires the METASPACE engine pipeline first
- The analysis requires strict annotation confidence (e.g., FDR <= 5-10%); this skill deliberately uses the most permissive FDR <= 50% threshold to capture matrix clusters, which would inflate false positives in high-confidence workflows
- The matrix is not DHB (e.g., CHCA, 9-AA) — the 353 cluster formulas are specific to 2,5-dihydroxybenzoic acid generated per the Keller and Li (2000) model

## Inputs

- CSV of generated DHB matrix-cluster molecular formulas (353 formulas, DHB mc mol formulas, predicted.csv)
- 31 DHB-positive-mode MALDI-imaging gold standard datasets in METASPACE
- HMDB v4 metabolite database (annotation reference in METASPACE)
- Gold standard ion images archive (GS.tar.gz)

## Outputs

- FDR<=50% filtered METASPACE annotations for the target formulas (6677 DHB cluster ion annotations)
- Per-ion-image classifier labels with predicted off-sample/on-sample probabilities
- Summary statistics table dhb_reannotation_summary.csv (6677 annotations; 3134 ~47% off-sample; ~5% of 58,203 total; 4.3% isomeric)

## How to apply

Load the generated matrix-cluster molecular formulas (353 DHB matrix clusters from the 'DHB mc mol formulas, predicted.csv' file, generated per the Keller and Li, 2000 combinatorial cluster model). Re-annotate the target datasets (31 DHB positive-mode gold standard datasets) in METASPACE using these formulas searched against HMDB v4, applying the False Positive Rate <= 50% cutoff, which is the most permissive METASPACE value and therefore the acceptance threshold for every annotation. Download and unpack the gold standard ion images archive (GS.tar.gz), then serve each annotated ion image through the best-performing ResNet50 CNN (pretrained on ImageNet, trained with fastai/PyTorch, deployed via the off-sample REST microservice) to obtain off-sample/on-sample labels with predicted probabilities. Finally, compute summary statistics (total target annotations, fraction off-sample, share of all annotations, fraction isomeric with non-matrix HMDB molecules) and export them to a headered CSV table such as dhb_reannotation_summary.csv.

## Related tools

- **METASPACE** (Cloud platform used to re-annotate the 31 DHB positive-mode datasets with DHB matrix-cluster formulas against HMDB v4 at FDR <= 50%, and source of the 58,203 total annotated ions) — https://github.com/metaspace2020/metaspace/
- **Fastai** (Library (v1.0.34) with which the best-performing ResNet50 off-sample classifier was trained; used to serve classifications for the FDR-filtered annotations) — https://github.com/metaspace2020/offsample
- **PyTorch** (Framework (v1.0) underlying the fastai training and the off-sample REST microservice inference on annotated ion images) — https://github.com/metaspace2020/offsample
- **torchvision** (Source (v0.2.1) of the ResNet-50 architecture pretrained on ImageNet used for off-sample recognition of the filtered annotations) — https://github.com/metaspace2020/offsample

## Examples

```
wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz
```

## Evaluation signals

- The reproduced count of DHB matrix cluster ion annotations must equal 6677 (all annotated at FDR <= 50% against HMDB v4)
- The number of cluster ions recognized as off-sample by the ResNet50 CNN must equal 3134 (~47%)
- The off-sample share of all 58,203 annotated ions must be approximately 5%
- The fraction of recognized cluster ions isomeric with non-matrix HMDB v4 molecules must equal 4.3%
- Every exported annotation in dhb_reannotation_summary.csv must fall within the FDR <= 50% acceptance threshold, and every ion image must carry a classifier label with a predicted probability

## Limitations

- The FDR <= 50% threshold is the most permissive METASPACE value, so annotations are deliberately low-confidence and only meaningful in combination with off-sample classification
- The off-sample classifier tags whole ion images, not pixels: the approach does not solve the segmentation problem of finding pixels in the off-sample region
- Ground truth about molecular content is lacking for imaging MS samples, so isomeric matches (4.3% with non-matrix HMDB molecules) cannot be fully disambiguated
- Deep learning inference depends on the deployed off-sample microservice; the trained model requires GPU resources and substantial training time (days), as described in Supplementary Data D1

## Evidence

- [discussion] Annotation filtering at False Positive Rate <= 50% in METASPACE when selecting ion images for tagging: "we considered annotations at a False Positive Rate < = 50% (the most permissive value in METASPACE)"
- [readme] DHB matrix cluster formulas generated per Keller and Li 2000: [evidence span withheld — anchor status: not_found]
- [readme] Gold standard ion images archive download: "wget -qO - https://github.com/metaspace2020/offsample/releases/download/0.2/GS.tar.gz | tar -xvz"
- [discussion] ResNet50 model source for off-sample classification: "with the ResNet-50 model from the torchvision library (v0.2.1)"
- [discussion] Deep learning outperforms other off-sample recognition approaches: "We discovered that deep learning outperforms other approaches in the F1 off-sample score"
