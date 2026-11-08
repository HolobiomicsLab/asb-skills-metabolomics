---
name: bgc-product-class-classification
description: Use when you have MIBiG-labeled BGC training sets in Pfam TSV format
  (e.g. biosynthetic-class with 1355 samples, chem-act with 370 samples) and need
  to predict product class or chemical activity per BGC via multi-label classification
  with AUC-ROC evaluation.
when_to_use_negative:
- The goal is BGC detection/localization within a genome sequence — that is handled
  by the DeepBGC BiLSTM detector, not the random forest classifier, which operates
  on already-detected BGCs' Pfam domain vectors.
- Input is raw nucleotide sequence rather than Pfam domain annotations — first run
  `deepbgc prepare` to produce the Pfam TSV training format before classification.
- BGCs from poorly characterized taxa outside the Streptomyces-biased MIBiG training
  distribution, where training-data bias limits label generalization.
edam_operation: http://edamontology.org/operation_3923
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0154
tools:
- name: DeepBGC
  role: Provides the `deepbgc train` command, the RandomForestClassifier JSON template
    with OneHotEncodingTransformer (sequence_as_vector=true), and the MIBiG-derived
    Pfam TSV training data downloads used for classifier training and cross-validation
  repo: https://github.com/Merck/deepbgc
- name: scikit-learn
  role: Supplies the multi-label RandomForestClassifier (n_estimators=100, random_state=0)
    and the val_auc_roc cross-validation metric
- name: antiSMASH
  role: 'Rule-based baseline: its product-type ROC point (AUC 0.78) from Table 1 is
    included in the cross-validation results comparison'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:cbcfea440be382558aa0fa95e088cf7543bb847fcc074ab5bbff1e4933e05778
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
  edam_operation: http://edamontology.org/operation_3923
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# bgc-product-class-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and cross-validate scikit-learn random forest classifiers that predict biosynthetic gene cluster (BGC) product class and molecular activity from Pfam domain composition, benchmarking against the antiSMASH product-type baseline (AUC 0.78).

## When to use

Use when you have MIBiG-labeled BGC training sets in Pfam TSV format (e.g. biosynthetic-class with 1355 samples, chem-act with 370 samples) and need to predict product class or chemical activity per BGC via multi-label classification with AUC-ROC evaluation.

## When NOT to use

- The goal is BGC detection/localization within a genome sequence — that is handled by the DeepBGC BiLSTM detector, not the random forest classifier, which operates on already-detected BGCs' Pfam domain vectors.
- Input is raw nucleotide sequence rather than Pfam domain annotations — first run `deepbgc prepare` to produce the Pfam TSV training format before classification.
- BGCs from poorly characterized taxa outside the Streptomyces-biased MIBiG training distribution, where training-data bias limits label generalization.

## Inputs

- MIBiG-derived biosynthetic-class training set in Pfam TSV format (1355 BGC samples, deepbgc release v0.1.0/v0.1.5)
- MIBiG-derived chem-act training set in Pfam TSV format (370 samples)
- RandomForestClassifier JSON template with OneHotEncodingTransformer (sequence_as_vector=true)

## Outputs

- Trained multi-label random forest classification model (product class and/or chem-act labels)
- 5-fold cross-validation AUC-ROC table per label (classifier_cv_results.csv) including antiSMASH baseline point (AUC 0.78)
- Per-BGC product class and molecular activity predictions when applied to detected BGCs

## How to apply

Download the MIBiG-derived Pfam TSV training sets (biosynthetic-class, 1355 BGC samples; chem-act, 370 samples) from the deepbgc releases (v0.1.0/v0.1.5). Encode each BGC's Pfam domain composition as a single binary feature vector using the OneHotEncodingTransformer with sequence_as_vector=true from the RandomForestClassifier JSON template. Train scikit-learn multi-label RandomForestClassifier models (n_estimators=100, random_state=0) via `deepbgc train` for both label sets, then run 5-fold cross-validation and compute per-label AUC ROC (val_auc_roc metric). Compare the resulting DeepBGC classifier AUCs against the antiSMASH product-type ROC point (AUC 0.78) from Table 1 and save the cross-validation table including the antiSMASH baseline as classifier_cv_results.csv. Validation: cross-validated AUCs reproduce the reported values (0.80 for product class, 0.61 for molecular activity) and the product-class classifier matches or exceeds the antiSMASH baseline of 0.78.

## Related tools

- **DeepBGC** (Provides the `deepbgc train` command, the RandomForestClassifier JSON template with OneHotEncodingTransformer (sequence_as_vector=true), and the MIBiG-derived Pfam TSV training data downloads used for classifier training and cross-validation) — https://github.com/Merck/deepbgc
- **scikit-learn** (Supplies the multi-label RandomForestClassifier (n_estimators=100, random_state=0) and the val_auc_roc cross-validation metric)
- **antiSMASH** (Rule-based baseline: its product-type ROC point (AUC 0.78) from Table 1 is included in the cross-validation results comparison)

## Examples

```
deepbgc train biosynthetic_class.pfam.tsv --model RandomForestClassifier --classifier-type cv --output classifier_cv_results
```

## Evaluation signals

- 5-fold cross-validation on MIBiG-labeled training data yields an average AUC of 0.80 for BGC product class prediction and 0.61 for molecular activity classification
- Product-class classifier AUC matches or exceeds the antiSMASH rule-based baseline AUC of 0.78
- Training sets have expected sizes: 1355 BGC samples for biosynthetic-class and 370 samples for chem-act
- Per-label val_auc_roc values are computed for every label in the multi-label output table (classifier_cv_results.csv)
- Classifier input vectors are binary one-hot encodings of Pfam domain composition (sequence_as_vector=true), so feature dimensionality equals the Pfam vocabulary size

## Limitations

- Training data derived from existing BGC databases is heavily biased towards natural product 'workhorses' such as Streptomyces, limiting classification performance on BGCs from poorly characterized bacteria.
- Molecular activity classification performs noticeably worse (AUC 0.61) than product class prediction (AUC 0.80), so activity predictions should be treated as putative.
- Random forest classification is a supplement to DeepBGC detection — some well-studied classes were recovered regardless of threshold while others were more threshold-dependent in the upstream detection step.

## Evidence

- [abstract] Random forest classifiers accurately predicted BGC product classes and potential chemical activity: "random forest classifiers that accurately predicted BGC product classes and potential chemical activity"
- [discussion] RF classification by Pfam composition: "The supplemental random forest classification approach allowed us to accurately identify BGC classes by their Pfam domain composition"
- [methods] Training data preparation format: "The training examples need to be prepared in Pfam TSV format, which can be prepared from your sequence using `deepbgc prepare`."
- [methods] Training own classification models: "You can train your own BGC detection and classification models, see `deepbgc train --help` for documentation and examples."
