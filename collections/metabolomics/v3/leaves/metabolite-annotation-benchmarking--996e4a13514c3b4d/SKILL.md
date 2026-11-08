---
name: metabolite-annotation-benchmarking
description: Use when candidate metabolite annotations must be filtered by predicted-vs-observed
  retention time agreement and you need an objective threshold chosen via ROC analysis
  on a labeled test set.
when_to_use_negative:
- The input is unlabeled (no way to mark candidates as true vs. false identities),
  since ROC analysis requires ground-truth labels on a test set.
- Predictions come from a retention time model other than RT-Transformer with substantially
  different error characteristics; the reported 5.7%/4.3% thresholds are tuned to
  this model's error distribution and must be re-derived, not reused.
- The chromatographic method differs from the training/transfer conditions without
  fine-tuning; the paper motivates the model precisely because different chromatographic
  conditions change retention times.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: RT-Transformer
  role: Graph-attention + 1D-Transformer deep learning model that produces predicted
    retention times whose agreement with observed RT drives the false-identity filter
    being benchmarked
  repo: https://github.com/01dadada/RT-Transformer
- name: scikit-learn (roc_curve/roc_auc_score)
  role: Used to sweep thresholds and build the ROC curves from which the optimal RT-error
    cutoffs are selected
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:58fcfde1e253cfe5ce462c7a0251190b6639434a75b60885cea08cb4ba34f690
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  - scikit-learn (roc_curve/roc_auc_score)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolite-annotation-benchmarking

## Summary

Benchmark a retention-time-based metabolite annotation filter by sweeping RT-error thresholds, building ROC curves on held-out test sets (SMRT and SMRT_Retained), and reporting optimal thresholds plus the fraction of false-positive identities eliminated.

## When to use

Use when candidate metabolite annotations must be filtered by predicted-vs-observed retention time agreement and you need an objective threshold chosen via ROC analysis on a labeled test set.

## When NOT to use

- The input is unlabeled (no way to mark candidates as true vs. false identities), since ROC analysis requires ground-truth labels on a test set.
- Predictions come from a retention time model other than RT-Transformer with substantially different error characteristics; the reported 5.7%/4.3% thresholds are tuned to this model's error distribution and must be re-derived, not reused.
- The chromatographic method differs from the training/transfer conditions without fine-tuning; the paper motivates the model precisely because different chromatographic conditions change retention times.

## Inputs

- SMRT and SMRT_Retained test sets with InChI structures and observed LC retention times (CSV with 'InChI' and 'RT' columns per the repo convention)
- RT-Transformer predicted retention times for each test molecule (produced by test.py or QuickStart.ipynb)
- Candidate identity labels (true/false) derivable from predicted-vs-observed RT agreement

## Outputs

- ROC curve points (true-positive rate vs. false-positive rate) per test set
- Optimal RT-error threshold per test set (5.7% for SMRT, 4.3% for SMRT_Retained)
- False-identity elimination rate at each optimal threshold (80.89% SMRT, 85.42% SMRT_Retained)
- rt_filter_roc.csv containing ROC curve and threshold metrics

## How to apply

1. Run the RT-Transformer model to produce predicted retention times for molecules in each test set (SMRT and SMRT_Retained) alongside the observed LC retention times. 2. For every test molecule compute the relative RT error between the RT-Transformer prediction and the observed retention time, and label candidate identities as true or false based on whether predicted RT agrees with the observed RT. 3. Sweep RT-error threshold values and, for each set, construct a ROC curve (true-positive rate vs. false-positive rate) with scikit-learn's roc_curve/roc_auc_score. 4. Select the optimal operating point on each curve — the paper reports 5.7% relative RT error for SMRT and 4.3% for SMRT_Retained — as the filter cutoff. 5. At each chosen threshold, compute the fraction of false identities removed and write the ROC points plus threshold metrics (e.g. rt_filter_roc.csv), verifying that elimination rates match 80.89% (SMRT) and 85.42% (SMRT_Retained). The rationale is that RT-Transformer's low average absolute error (27.30 on SMRT after removing non-retained molecules) makes small RT-error cutoffs effective discriminators of false positives.

## Related tools

- **RT-Transformer** (Graph-attention + 1D-Transformer deep learning model that produces predicted retention times whose agreement with observed RT drives the false-identity filter being benchmarked) — https://github.com/01dadada/RT-Transformer
- **scikit-learn (roc_curve/roc_auc_score)** (Used to sweep thresholds and build the ROC curves from which the optimal RT-error cutoffs are selected)

## Examples

```
python ./test.py; sklearn.metrics.roc_curve(y_true, rel_rt_err) — sweep rel_rt_err thresholds, then write rt_filter_roc.csv
```

## Evaluation signals

- Reproduced optimal thresholds match the reported values: 5.7% relative RT error on SMRT and 4.3% on SMRT_Retained.
- Reproduced false-identity elimination rates match the reported values: 80.89% on SMRT and 85.42% on SMRT_Retained within numerical tolerance.
- ROC AUC computed with scikit-learn roc_auc_score is consistent with a discriminative filter (well above 0.5) on both test sets.
- rt_filter_roc.csv contains monotonic ROC points (TPR non-decreasing in FPR) for both SMRT and SMRT_Retained curves.
- Re-running test.py on the SMRT/SMRT_Retained splits yields predicted RTs whose relative-error distribution is stable across runs.

## Limitations

- The reported thresholds (5.7% and 4.3%) were derived on the SMRT and SMRT_Retained test sets; they are specific to RT-Transformer's error distribution and the paper's split and must be re-estimated for other models or splits.
- SMRT dataset contains non-retained molecules; the paper reports an average absolute error of 27.30 only after removing not-retained molecules (33.41 when all samples are kept), so error behavior differs depending on whether non-retained molecules are filtered first.
- Transferability across chromatographic conditions depends on fine-tuning; the model was pre-trained on the 80,038-molecule SMRT dataset and then transferred to 5 chromatographic-condition datasets from PredRet, so retention-time predictions may degrade for unseen chromatographic methods.

## Evidence

- [results] Optimal thresholds derived on labeled test sets: [evidence span withheld — anchor status: not_found]
- [results] Filter effectiveness claim: [evidence span withheld — anchor status: not_found]
- [abstract] Model error characteristics: [evidence span withheld — anchor status: not_found]
- [abstract] Chromatographic-method dependence: "different chromatographic conditions may result in different retention times for the same metabolite"
- [readme] Repository usage entry point: [evidence span withheld — anchor status: external_doc]
- [readme] Input format convention: [evidence span withheld — anchor status: external_doc]
