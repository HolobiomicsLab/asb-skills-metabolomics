---
name: cross-validated-auc-benchmarking
description: Use when evaluating BGC classification models on MIBiG-labeled Pfam TSV
  data and you need cross-validated AUC ROC values to compare machine-learned classifiers
  against an antiSMASH rule-based baseline.
when_to_use_negative:
- The goal is BGC detection (in_cluster classification of Pfam domains within genomes)
  rather than assigning product class or chemical activity to already-detected BGCs
- Input is raw genome sequence (FASTA) rather than pre-annotated Pfam TSV training
  examples — run `deepbgc prepare` first or use the detection pipeline instead
- No labeled training data with MIBiG product-class or activity annotations is available,
  making supervised cross-validated AUC estimation impossible
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: DeepBGC
  role: Provides the `deepbgc train` command used to train and cross-validate the
    random forest classifiers on MIBiG-labeled Pfam TSV sets and compute val_auc_roc
  repo: https://github.com/Merck/deepbgc
- name: antiSMASH
  role: Rule-based baseline whose product-type ROC point (AUC 0.78) serves as the
    external benchmark in the comparison table
- name: scikit-learn
  role: Supplies the multi-label RandomForestClassifier (n_estimators=100, random_state=0)
    used for product-class and activity prediction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:99e4a63b7873f79b358184077295b44a45e53af1d48f2f9b584699ddc18ee8ee
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Merck/deepbgc
  repo_url: https://github.com/Merck/deepbgc
  tools:
  - DeepBGC
  - antiSMASH
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cross-validated-auc-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark random-forest BGC product-class and molecular-activity classifiers with 5-fold cross-validated AUC ROC on MIBiG-derived Pfam training sets, comparing DeepBGC classifier performance against the antiSMASH rule-based baseline (AUC 0.78).

## When to use

Use when evaluating BGC classification models on MIBiG-labeled Pfam TSV data and you need cross-validated AUC ROC values to compare machine-learned classifiers against an antiSMASH rule-based baseline.

## When NOT to use

- The goal is BGC detection (in_cluster classification of Pfam domains within genomes) rather than assigning product class or chemical activity to already-detected BGCs
- Input is raw genome sequence (FASTA) rather than pre-annotated Pfam TSV training examples — run `deepbgc prepare` first or use the detection pipeline instead
- No labeled training data with MIBiG product-class or activity annotations is available, making supervised cross-validated AUC estimation impossible

## Inputs

- biosynthetic-class Pfam TSV training set (1355 BGC samples, MIBiG labels, deepbgc release v0.1.0/v0.1.5)
- chem-act Pfam TSV training set (370 BGC samples, MIBiG labels, deepbgc release v0.1.0/v0.1.5)
- RandomForestClassifier JSON model template with OneHotEncodingTransformer (sequence_as_vector=true)

## Outputs

- classifier_cv_results.csv — 5-fold cross-validation AUC ROC per label (val_auc_roc) for biosynthetic-class and chem-act models
- Trained multi-label RandomForestClassifier models (n_estimators=100, random_state=0)
- antiSMASH baseline AUC point (0.78) merged into the comparison table

## How to apply

Download the MIBiG-derived Pfam TSV training sets from deepbgc releases v0.1.0/v0.1.5 — biosynthetic-class (1355 BGC samples) for product-class labels and chem-act (370 samples) for molecular-activity labels. Encode each BGC's Pfam domain composition as a single binary vector using the OneHotEncodingTransformer with sequence_as_vector=true from the RandomForestClassifier JSON template. Train scikit-learn multi-label RandomForestClassifier models (n_estimators=100, random_state=0) via `deepbgc train` for both label sets, then run 5-fold cross-validation and compute the per-label val_auc_roc metric. Compare the resulting AUCs against the antiSMASH product-type ROC point (AUC 0.78) from Table 1, and save the cross-validation AUC table including the antiSMASH baseline as classifier_cv_results.csv.

## Related tools

- **DeepBGC** (Provides the `deepbgc train` command used to train and cross-validate the random forest classifiers on MIBiG-labeled Pfam TSV sets and compute val_auc_roc) — https://github.com/Merck/deepbgc
- **antiSMASH** (Rule-based baseline whose product-type ROC point (AUC 0.78) serves as the external benchmark in the comparison table)
- **scikit-learn** (Supplies the multi-label RandomForestClassifier (n_estimators=100, random_state=0) used for product-class and activity prediction)

## Examples

```
deepbgc train --help
```

## Evaluation signals

- 5-fold cross-validation on MIBiG-labeled data reproduces the Table 1 reported AUCs: average AUC 0.80 for BGC product-class prediction and AUC 0.61 for molecular-activity prediction
- The product-class classifier AUC matches or exceeds the antiSMASH baseline AUC of 0.78 from Table 1
- The OneHotEncodingTransformer uses sequence_as_vector=true so each BGC's Pfam domain composition is encoded as a single binary vector before training
- classifier_cv_results.csv contains per-label val_auc_roc values for both biosynthetic-class and chem-act label sets plus the antiSMASH baseline point
- Training set sizes match expectations: 1355 BGC samples for biosynthetic-class and 370 for chem-act

## Limitations

- Training data is heavily biased towards BGCs from natural product 'workhorses' such as Streptomyces, limiting classifier generalization
- Molecular-activity classification is weak (AUC 0.61), near the boundary of useful discrimination
- Further work is needed to curate more diverse BGC databases which can be used to improve the training and validation of these classifiers

## Evidence
