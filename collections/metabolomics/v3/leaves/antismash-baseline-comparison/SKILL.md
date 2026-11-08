---
name: antismash-baseline-comparison
description: Use when evaluating a BGC product-class or chemical-activity classifier
  on MIBiG-labeled Pfam TSV training data and you need the antiSMASH rule-based product-type
  AUC (0.78) as a comparative baseline in a 5-fold cross-validation table.
when_to_use_negative:
- Input is raw genomic FASTA requiring BGC detection rather than pre-labeled BGC feature
  vectors — the `deepbgc pipeline` detection workflow, not classifier benchmarking,
  is the right path
- You need a full antiSMASH ROC curve or score-distributed comparison — antiSMASH
  only supplies a single product-type ROC point (AUC 0.78) in this evaluation
- The goal is BGC boundary detection accuracy (e.g., ClusterFinder comparison) rather
  than product-class or chemical-activity classification
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: DeepBGC
  role: Provides `deepbgc train` for training the random forest classifiers and the
    Pfam TSV training data (biosynthetic-class, chem-act) used for cross-validated
    AUC computation
  repo: https://github.com/Merck/deepbgc
- name: antiSMASH
  role: Supplies the rule-based product-type classification ROC point (AUC 0.78) used
    as the comparison baseline in Table 1
- name: scikit-learn
  role: Backend RandomForestClassifier implementation (n_estimators=100, random_state=0)
    trained per label set and evaluated with val_auc_roc
- name: bgc-pipeline
  role: Repository containing the evaluation code and data for the cross-validation
    comparison against ClusterFinder and antiSMASH
  repo: https://github.com/Merck/bgc-pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:0738d51d5d16b9e0e04283e38e1323f9c6075fa273902fc4db1678c6ae0e9c70
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
  - bgc-pipeline
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# antismash-baseline-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark DeepBGC's random forest BGC classifiers (product class and chemical activity) against the antiSMASH rule-based product-type classifier by comparing cross-validated AUC ROC values, including the antiSMASH ROC point (AUC 0.78) from Table 1 as a baseline column.

## When to use

Use when evaluating a BGC product-class or chemical-activity classifier on MIBiG-labeled Pfam TSV training data and you need the antiSMASH rule-based product-type AUC (0.78) as a comparative baseline in a 5-fold cross-validation table.

## When NOT to use

- Input is raw genomic FASTA requiring BGC detection rather than pre-labeled BGC feature vectors — the `deepbgc pipeline` detection workflow, not classifier benchmarking, is the right path
- You need a full antiSMASH ROC curve or score-distributed comparison — antiSMASH only supplies a single product-type ROC point (AUC 0.78) in this evaluation
- The goal is BGC boundary detection accuracy (e.g., ClusterFinder comparison) rather than product-class or chemical-activity classification

## Inputs

- MIBiG-derived biosynthetic-class training set in Pfam TSV format (1355 BGC samples, deepbgc release v0.1.0/v0.1.5)
- MIBiG-derived chem-act training set in Pfam TSV format (370 BGC samples)
- antiSMASH product-type ROC point (AUC 0.78) from Table 1 of the DeepBGC paper

## Outputs

- classifier_cv_results.csv containing per-label cross-validated AUC ROC values plus the antiSMASH baseline point
- Trained multi-label RandomForestClassifier models for biosynthetic-class and chem-act label sets

## How to apply

Download the MIBiG-derived training sets in Pfam TSV format (biosynthetic-class, 1355 BGC samples; chem-act, 370 samples) from the deepbgc releases v0.1.0/v0.1.5. Encode each BGC's Pfam domain composition as a single binary vector using the OneHotEncodingTransformer with sequence_as_vector=true, then train scikit-learn multi-label RandomForestClassifier models (n_estimators=100, random_state=0) via `deepbgc train`. Run 5-fold cross-validation and compute the per-label val_auc_roc metric. Add the antiSMASH product-type ROC point (AUC 0.78 from Table 1) as a separate baseline row/column, since antiSMASH is a rule-based approach that yields a point rather than a score-distributed curve. Save the combined cross-validation AUC table (classifier_cv_results.csv) and check that DeepBGC product-class AUC (0.80) matches or exceeds the 0.78 antiSMASH baseline while the chem-act classifier reaches ~0.61.

## Related tools

- **DeepBGC** (Provides `deepbgc train` for training the random forest classifiers and the Pfam TSV training data (biosynthetic-class, chem-act) used for cross-validated AUC computation) — https://github.com/Merck/deepbgc
- **antiSMASH** (Supplies the rule-based product-type classification ROC point (AUC 0.78) used as the comparison baseline in Table 1)
- **scikit-learn** (Backend RandomForestClassifier implementation (n_estimators=100, random_state=0) trained per label set and evaluated with val_auc_roc)
- **bgc-pipeline** (Repository containing the evaluation code and data for the cross-validation comparison against ClusterFinder and antiSMASH) — https://github.com/Merck/bgc-pipeline

## Examples

```
deepbgc train --help
```

## Evaluation signals

- Cross-validated AUC for the biosynthetic-class random forest averages 0.80 across 5 folds
- Cross-validated AUC for the chem-act random forest averages ~0.61
- The product-class classifier AUC (0.80) matches or exceeds the antiSMASH baseline AUC of 0.78
- classifier_cv_results.csv contains one row/entry per label plus the antiSMASH product-type ROC point
- Re-running with random_state=0 and n_estimators=100 reproduces the per-label val_auc_roc values

## Limitations

- The antiSMASH comparison is against a single ROC point (AUC 0.78), not a full ROC curve, because antiSMASH is a rule-based classifier
- Training data from existing BGC databases is heavily biased towards BGCs from natural product 'workhorses' such as Streptomyces
- Some BGC product classes were mis-identified or threshold-dependent in validation examples such as the S. coelicolor reference genome
- The majority of novel BGC classes identified could not be confidently assigned

## Evidence

- [abstract] Random forest product-class and activity classifiers supplementing DeepBGC detection: "We supplemented this with random forest classifiers that accurately predicted BGC product classes and potential chemical activity"
- [discussion] antiSMASH as rule-based baseline: "the comparison of DeepBGC to antiSMASH, which represents a rule-based approach to BGC identification"
- [discussion] Pfam domain composition drives classification: "The supplemental random forest classification approach allowed us to accurately identify BGC classes by their Pfam domain composition"
- [methods] Training classifiers with deepbgc train: "You can train your own BGC detection and classification models, see `deepbgc train --help` for documentation and examples."
- [readme] bgc-pipeline provides evaluation code against antiSMASH: [evidence span withheld — anchor status: external_doc]
