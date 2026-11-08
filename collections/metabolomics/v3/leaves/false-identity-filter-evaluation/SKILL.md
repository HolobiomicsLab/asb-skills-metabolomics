---
name: false-identity-filter-evaluation
description: Use when you have observed vs. model-predicted retention times for candidate
  metabolites on SMRT or SMRT_Retained test sets and must quantify how many false
  identities an RT-error threshold removes.
when_to_use_negative:
- When the input is a feature table or peak list without observed and predicted retention
  times per candidate — there is no RT error to threshold.
- When evaluating chromatographic conditions for which the RT-Transformer model has
  not been fine-tuned; the reported thresholds are specific to the SMRT and SMRT_Retained
  test sets.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3174
tools:
- name: RT-Transformer
  role: Generates the predicted retention times whose disagreement with observed RTs
    defines true vs. false identities for the ROC sweep
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:42e582ffebb935f04057a597408cc6a2f863c8d08615119611cd4ca468521fb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# false-identity-filter-evaluation

## Summary

Evaluates how well a retention-time-based filter removes false candidate compounds by sweeping relative RT-error thresholds, building ROC curves on SMRT and SMRT_Retained test sets, and reporting optimal thresholds and false-identity elimination rates.

## When to use

Use when you have observed vs. model-predicted retention times for candidate metabolites on SMRT or SMRT_Retained test sets and must quantify how many false identities an RT-error threshold removes.

## When NOT to use

- When the input is a feature table or peak list without observed and predicted retention times per candidate — there is no RT error to threshold.
- When evaluating chromatographic conditions for which the RT-Transformer model has not been fine-tuned; the reported thresholds are specific to the SMRT and SMRT_Retained test sets.

## Inputs

- SMRT test set with observed and RT-Transformer-predicted retention times (InChI / RT columns)
- SMRT_Retained test set with observed and RT-Transformer-predicted retention times
- RT-Transformer pretrained model weights (best_state_dict.pth / best_state_download_dict.pth)

## Outputs

- ROC curve points (true-positive rate vs. false-positive rate) for SMRT and SMRT_Retained test sets
- Optimal RT-error thresholds (5.7% for SMRT, 4.3% for SMRT_Retained)
- False-identity elimination rates at each optimal threshold (80.89% and 85.42%)
- rt_filter_roc.csv with curve points and threshold metrics for both test sets

## How to apply

Load the SMRT and SMRT_Retained test sets with observed RTs and RT-Transformer-predicted RTs from the 01dadada/RT-Transformer repository. For each test molecule compute the relative RT error and label candidates as true or false identities according to agreement between predicted and observed RT. Sweep RT-error threshold values and build ROC curves (true-positive rate vs. false-positive rate) separately for each test set using scikit-learn's roc_curve/roc_auc_score, then identify the optimal operating threshold on each curve — the reproduced values must match the reported 5.7% for SMRT and 4.3% for SMRT_Retained. At each optimal threshold compute the fraction of false identities removed, verifying 80.89% for SMRT and 85.42% for SMRT_Retained, and write the ROC curve points plus threshold metrics for both sets to rt_filter_roc.csv.

## Related tools

- **RT-Transformer** (Generates the predicted retention times whose disagreement with observed RTs defines true vs. false identities for the ROC sweep) — https://github.com/01dadada/RT-Transformer

## Examples

```
python ./test.py
```

## Evaluation signals

- Reproduced optimal threshold on the SMRT test set equals 5.7%
- Reproduced optimal threshold on the SMRT_Retained test set equals 4.3%
- Reproduced false-identity elimination rate at the SMRT optimal threshold equals 80.89%
- Reproduced false-identity elimination rate at the SMRT_Retained optimal threshold equals 85.42%
- rt_filter_roc.csv exists and contains ROC curve points and threshold metrics for both test sets

## Limitations

- Thresholds are validated only against the reported SMRT (5.7%) and SMRT_Retained (4.3%) values; other chromatographic conditions are not covered.
- Requires the pretrained RT-Transformer weights and the corresponding test sets from the RT-Transformer repository; unavailable data prevents reproduction.

## Evidence

- [methods] ROC optimal thresholds and elimination rates: [evidence span withheld — anchor status: not_found]
- [readme] Test procedure from README: [evidence span withheld — anchor status: external_doc]
