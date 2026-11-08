---
name: roc-curve-analysis
description: Use when a binary filter (e.g. RT-error agreement) must be tuned to a
  threshold that maximizes true-positive retention while quantifying false-identity
  removal, with ROC curves built via scikit-learn.
when_to_use_negative:
- The task is pure RT prediction (e.g. fine-tuning RT-Transformer on a new chromatographic
  method via transfer.py) rather than threshold calibration for identity filtering
  — then ROC analysis is premature.
- Candidate compounds have no observed retention time to compare against, so relative
  RT error cannot be computed and a ROC sweep is not defined.
- The filter criterion is not a continuous score threshold (e.g. a hard MS/MS spectral-match
  rule), for which a ROC curve over that score is not meaningful.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: RT-Transformer
  role: Produces the predicted retention times (from graph + fingerprint features
    of each molecule) whose relative error against observed RT defines the score swept
    in the ROC analysis
  repo: https://github.com/01dadada/RT-Transformer
- name: scikit-learn
  role: Provides roc_curve and roc_auc_score used to build the ROC curves and identify
    the optimal threshold on SMRT and SMRT_Retained
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:5c8bb8368d38848e8931cd8c557eb82de5aefb1338d40533a30e5f15b1807e4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# roc-curve-analysis

## Summary

Construct receiver operating characteristic (ROC) curves by sweeping a molecular-feature threshold (here, relative retention-time error between RT-Transformer-predicted and observed RT) to select an optimal operating point and quantify how many false-identity candidates the filter eliminates on held-out test sets such as SMRT and SMRT_Retained.

## When to use

Use when a binary filter (e.g. RT-error agreement) must be tuned to a threshold that maximizes true-positive retention while quantifying false-identity removal, with ROC curves built via scikit-learn.

## When NOT to use

- The task is pure RT prediction (e.g. fine-tuning RT-Transformer on a new chromatographic method via transfer.py) rather than threshold calibration for identity filtering — then ROC analysis is premature.
- Candidate compounds have no observed retention time to compare against, so relative RT error cannot be computed and a ROC sweep is not defined.
- The filter criterion is not a continuous score threshold (e.g. a hard MS/MS spectral-match rule), for which a ROC curve over that score is not meaningful.

## Inputs

- Test set with observed retention times and RT-Transformer-predicted retention times per molecule (e.g. SMRT, SMRT_Retained), keyed by InChI
- Relative RT error values per candidate compound (predicted vs. observed RT)
- True/false identity labels per candidate (derived from RT agreement)

## Outputs

- ROC curve points (true-positive rate vs. false-positive rate) per test set, e.g. written to rt_filter_roc.csv
- Optimal relative RT-error threshold per test set (reported: 5.7% for SMRT, 4.3% for SMRT_Retained)
- Fraction of false identities eliminated at the optimal threshold (reported: 80.89% SMRT, 85.42% SMRT_Retained)

## How to apply

Load the held-out test set (SMRT or SMRT_Retained) containing observed retention times and RT-Transformer-predicted retention times, computed from InChI structures via the RT-Transformer model. For each candidate compound, compute the relative RT error between the predicted and observed retention times and label it as a true or false identity according to whether that error exceeds a candidate threshold. Sweep thresholds using scikit-learn's `roc_curve` and `roc_auc_score` to trace the true-positive-rate vs. false-positive-rate curve for each test set separately. Select the optimal operating point — for this filter the reported optima are 5.7% relative RT error on SMRT and 4.3% on SMRT_Retained — then compute the fraction of false identities eliminated at that threshold, verifying the reported removal rates of 80.89% (SMRT) and 85.42% (SMRT_Retained). Write the ROC points and per-threshold metrics to a table such as rt_filter_roc.csv so downstream metabolite-annotation pipelines can reuse the calibrated cutoffs.

## Related tools

- **RT-Transformer** (Produces the predicted retention times (from graph + fingerprint features of each molecule) whose relative error against observed RT defines the score swept in the ROC analysis) — https://github.com/01dadada/RT-Transformer
- **scikit-learn** (Provides roc_curve and roc_auc_score used to build the ROC curves and identify the optimal threshold on SMRT and SMRT_Retained)

## Examples

```
python ./test.py  # compute predicted RTs, sweep relative RT-error thresholds, write rt_filter_roc.csv via roc_curve/roc_auc_score
```

## Evaluation signals

- Reproduced optimal thresholds match the reported 5.7% (SMRT) and 4.3% (SMRT_Retained) within numerical tolerance
- Reproduced false-identity elimination rates match the reported 80.89% (SMRT) and 85.42% (SMRT_Retained)
- ROC curve files (e.g. rt_filter_roc.csv) contain monotone TPR/FPR pairs for both test sets and separate curves per test set
- RT-Transformer-predicted RTs on the test sets reproduce the reported average absolute errors (27.30 after removing not-retained molecules; 33.41 with no samples removed) as a sanity check on the underlying predictions
- Threshold sweep covers the full score range so the reported optimum is a genuine maximum, not an artifact of a truncated grid

## Limitations

- Optimal thresholds are specific to the SMRT and SMRT_Retained test sets and their chromatographic conditions; other chromatographic methods may yield different optimal relative RT-error cutoffs.
- Filtering depends on the quality of RT-Transformer predictions; mispredictions (e.g. for not-retained molecules, which inflate average absolute error to 33.41 vs. 27.30 when removed) directly degrade the ROC separation.
- The ROC analysis presupposes labeled true/false identities, which themselves derive from RT agreement — circularity risk if labels are not independently validated.

## Evidence

- [results] ROC curves on the SMRT and SMRT_Retained test sets yielded optimal thresholds of 5.7% and 4.3%, which eliminated 80.89% and 85.42% of false identities, respectively: [evidence span withheld — anchor status: not_found]
- [methods] Sweep RT-error threshold values and build ROC curves ... using scikit-learn's roc_curve/roc_auc_score: [evidence span withheld — anchor status: not_found]
- [abstract] the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
- [readme] Load the SMRT and SMRT_Retained test sets with observed and RT-Transformer-predicted retention times from the RT-Transformer repository: [evidence span withheld — anchor status: not_found]
