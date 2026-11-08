---
name: retention-time-error-threshold-optimization
description: Use when candidate compounds from untargeted metabolomics have RT-Transformer-predicted
  vs observed RTs and you must choose an RT-error cutoff to discard false identities
  without losing true ones.
when_to_use_negative:
- The task is RT prediction itself (training/fine-tuning RT-Transformer) rather than
  selecting a filtering threshold from already-predicted RTs
- There are no labeled true/false identity candidates, so no ROC curve can be built
  and the threshold cannot be validated
- The data are gas chromatography retention indices or other non-liquid-chromatography
  RT data outside the RT-Transformer LC scope
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: RT-Transformer
  role: Provides the predicted retention times for each test molecule whose relative
    RT error is swept to build the ROC curves and pick the optimal threshold
  repo: https://github.com/01dadada/RT-Transformer
- name: scikit-learn roc_curve/roc_auc_score
  role: Computes ROC curve points and AUC when sweeping RT-error thresholds on the
    SMRT and SMRT_Retained test sets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:9c489deb2fdee7a2656429aff4e1da05e991b68b78d05c4cb8d85afaa34dc8f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  - scikit-learn roc_curve/roc_auc_score
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# retention-time-error-threshold-optimization

## Summary

Sweeps relative retention-time (RT) error thresholds on ROC curves built from the SMRT and SMRT_Retained test sets to pick the cutoff that maximizes elimination of false-positive candidate compounds when filtering metabolite annotations with RT-Transformer predictions.

## When to use

Use when candidate compounds from untargeted metabolomics have RT-Transformer-predicted vs observed RTs and you must choose an RT-error cutoff to discard false identities without losing true ones.

## When NOT to use

- The task is RT prediction itself (training/fine-tuning RT-Transformer) rather than selecting a filtering threshold from already-predicted RTs
- There are no labeled true/false identity candidates, so no ROC curve can be built and the threshold cannot be validated
- The data are gas chromatography retention indices or other non-liquid-chromatography RT data outside the RT-Transformer LC scope

## Inputs

- Test set of molecules with observed retention times and RT-Transformer-predicted retention times (e.g. SMRT test set, SMRT_Retained test set)
- Labels marking each candidate compound as a true or false identity based on RT agreement
- scikit-learn roc_curve/roc_auc_score functions for the ROC sweep

## Outputs

- ROC curve points (true-positive rate vs false-positive rate) per test set
- Optimal RT-error threshold values (e.g. 5.7% for SMRT, 4.3% for SMRT_Retained)
- Fraction of false identities eliminated at each optimal threshold (e.g. 80.89% and 85.42%)
- CSV file of ROC curve points and threshold metrics (e.g. rt_filter_roc.csv)

## How to apply

Load the SMRT and SMRT_Retained test sets from the RT-Transformer repository and compute the relative RT error (observed minus RT-Transformer-predicted retention time) for each molecule, labeling candidates as true or false identities based on whether the predicted RT agrees with the observed RT. Sweep RT-error threshold values and build ROC curves (true-positive rate vs false-positive rate) separately for each test set using scikit-learn's roc_curve/roc_auc_score; identify the optimal operating threshold on each curve, which the paper reports as 5.7% for SMRT and 4.3% for SMRT_Retained. At the chosen threshold, compute the fraction of false identities removed, verifying the reported 80.89% (SMRT) and 85.42% (SMRT_Retained) elimination rates. The rationale is that a tighter cutoff (lower % RT error tolerance) discards more false positives, but the optimum must be picked from the ROC so true identities are not sacrificed; writing the ROC points and threshold metrics to a CSV (e.g. rt_filter_roc.csv) makes the sweep auditable.

## Related tools

- **RT-Transformer** (Provides the predicted retention times for each test molecule whose relative RT error is swept to build the ROC curves and pick the optimal threshold) — https://github.com/01dadada/RT-Transformer
- **scikit-learn roc_curve/roc_auc_score** (Computes ROC curve points and AUC when sweeping RT-error thresholds on the SMRT and SMRT_Retained test sets)

## Examples

```
python ./test.py  # load SMRT/SMRT_Retained test sets, compute RT errors, sweep thresholds, and write rt_filter_roc.csv
```

## Evaluation signals

- Reproduced optimal thresholds must match the reported 5.7% (SMRT) and 4.3% (SMRT_Retained) within numerical tolerance
- Reproduced false-identity elimination rates must match the reported 80.89% (SMRT) and 85.42% (SMRT_Retained) within numerical tolerance
- ROC curve must be built per test set (SMRT and SMRT_Retained separately), not pooled
- Threshold metrics should be exported to rt_filter_roc.csv for auditability

## Limitations

- Thresholds are chromatographic-condition specific — the paper's 5.7% and 4.3% values apply to the SMRT and SMRT_Retained test sets and may not transfer to other LC methods or transfer-learned datasets
- The model used to generate predicted RTs is the pre-trained RT-Transformer (best_state_dict.pth for full data, best_state_download_dict.pth for retained data), so retraining or fine-tuning changes the optimal threshold
- Elimination of false positives trades off against retention of true identities; the ROC operating point must be re-picked if the annotation candidate list changes

## Evidence

- [other] ROC curves on the SMRT and SMRT_Retained test sets yielded optimal thresholds of 5.7% and 4.3%, which eliminated 80.89% and 85.42% of false identities: [evidence span withheld — anchor status: not_found]
- [abstract] average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
- [abstract] RT-Transformer has excellent scalability in predicting retention times for liquid chromatography and improves the accuracy of metabolite identification: "Extensive evaluations indicate that RT-Transformer has excellent scalability in predicting retention times for liquid chromatography and improves the accuracy of metabolite identification"
- [readme] The source code for the model is available at https://github.com/01dadada/RT-Transformer: [evidence span withheld — anchor status: external_doc]
