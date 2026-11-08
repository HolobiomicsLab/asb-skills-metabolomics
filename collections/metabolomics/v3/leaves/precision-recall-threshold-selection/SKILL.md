---
name: precision-recall-threshold-selection
description: Use when you have a CCS predictor's output with experimental CCS references
  and need to discard predictions likely to exceed a 5% relative-error threshold,
  improving filtered-set MAE/MRE.
when_to_use_negative:
- You lack ground-truth CCS values for the test set, so the 5% error labels needed
  to train the confidence model cannot be constructed.
- The predictions come from a model architecture other than Mol2CCS/SigmaCCS-style
  GNNs whose feature representation the classifier was tuned on in this study.
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3316
tools:
- name: Mol2CCS
  role: Source of the CCS predictions whose reliability the confidence model assesses;
    its prediction features feed the classifier.
  repo: github.com/enveda/ccs-prediction
- name: Python
  role: Runtime for scikit-learn-based random forest training and threshold sweeps.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:51454508e4455304c5a48bd63ed7024487a01336eb7f1a749f3e11e115b3f644
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mol2CCS
  - Python
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# precision-recall-threshold-selection

## Summary

Train a confidence model (random forest classifier) that labels Mol2CCS CCS predictions as reliable or unreliable, sweep its probability threshold against a precision–recall curve, and select the operating point that filters out predictions with >5% CCS error to improve MAE/MRE.

## When to use

Use when you have a CCS predictor's output with experimental CCS references and need to discard predictions likely to exceed a 5% relative-error threshold, improving filtered-set MAE/MRE.

## When NOT to use

- You lack ground-truth CCS values for the test set, so the 5% error labels needed to train the confidence model cannot be constructed.
- The predictions come from a model architecture other than Mol2CCS/SigmaCCS-style GNNs whose feature representation the classifier was tuned on in this study.

## Inputs

- Mol2CCS prediction outputs (predicted CCS values) for test sets in each training regime (train-METLIN-test-METLIN, train-CCSBase-test-CCSBase, cross-dataset)
- Ground-truth experimental CCS values for the same test molecules (METLIN-CCS or CCSBase v1.3 records)
- Binary error labels derived from the 5% absolute relative error criterion

## Outputs

- Precision–recall curve over probability thresholds for the confidence classifier
- Selected confidence probability threshold for the high-confidence filter
- Filtered prediction subset (high-confidence CCS predictions)
- Filtered vs unfiltered MAE and MRE per training regime

## How to apply

Label each Mol2CCS prediction as a positive if its absolute relative error versus the true CCS value is within 5%; negatives are predictions exceeding that error. Train a random forest classifier (scikit-learn) on these binary labels using the model's prediction features, mirroring the paper's confidence-model approach. Sweep the classifier's probability threshold to map precision against recall and select the operating threshold for the high-confidence filter. Retain only predictions whose confidence score exceeds the threshold and recompute MAE and MRE on the filtered subset for each regime (train-METLIN-test-METLIN, train-CCSBase-test-CCSBase, and cross-dataset). Repeat with increasing in-domain training sample sizes; the paper shows MAE and MRE decrease as in-domain sample size grows, reproducing the pattern of Fig. 5 and Supplementary Figs. 9–13.

## Related tools

- **Mol2CCS** (Source of the CCS predictions whose reliability the confidence model assesses; its prediction features feed the classifier.) — github.com/enveda/ccs-prediction
- **Python** (Runtime for scikit-learn-based random forest training and threshold sweeps.)

## Examples

```
python scripts/train-test.py --prefix "train-metlin-test-metlin" --train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/metlin_test_3d.parquet" --model-output-file "model/train-metlin-test-metlin.h5"
```

## Evaluation signals

- Filtered-subset MAE and MRE must be lower than unfiltered MAE/MRE for each training regime.
- The precision–recall curve should show a trade-off where increasing the threshold raises precision at the cost of recall.
- MAE/MRE improvement should grow as in-domain training sample size increases, matching the paper's trend.
- Direction of improvement must match Fig. 5 and Supplementary Figs. 9–13 of the original study.

## Limitations

- Performance depends on the amount of in-domain training data; generalization declines in structurally novel chemical spaces, so the confidence model may not transfer across datasets.
- The 5% error threshold is specific to this study's CCS accuracy criterion and may not suit other analytical goals.
- Evaluation was restricted mainly to small molecules and synthetic structures (e.g., METLIN-CCS), limiting external validity.

## Evidence

- [other] Label Mol2CCS predictions as positive if the absolute relative error is within 5% of the true CCS value: [evidence span withheld — anchor status: not_found]
- [other] Filtering by confidence model improves MAE/MRE: [evidence span withheld — anchor status: not_found]
- [other] Train random forest classifier on prediction features: [evidence span withheld — anchor status: not_found]
- [other] Sweep probability threshold to map precision against recall: [evidence span withheld — anchor status: not_found]
- [other] Validation criterion: filtered MAE/MRE must be lower: [evidence span withheld — anchor status: not_found]
