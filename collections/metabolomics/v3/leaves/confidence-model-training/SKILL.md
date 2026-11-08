---
name: confidence-model-training
description: Use when GNN-predicted CCS values (e.g. Mol2CCS outputs on METLIN-CCS
  or CCSBase) must be filtered for reliability, and per-prediction absolute relative
  error is available on a labeled subset to train a confidence classifier.
when_to_use_negative:
- Predictions already come with calibrated per-sample uncertainty (e.g. Monte Carlo
  dropout variance alone) and no labeled residuals exist to supervise a classifier.
- The task is plain CCS prediction accuracy rather than reliability filtering — use
  Mol2CCS training/splitting workflows instead.
- Data comes from a chemical space far outside METLIN-CCS or CCSBase; the paper shows
  model accuracy declines on structurally novel regions, so filtered scores may not
  transfer.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3336
tools:
- name: Mol2CCS
  role: GNN (extended SigmaCCS) whose CCS predictions are labeled, featureized and
    filtered by the confidence model
  repo: https://github.com/enveda/ccs-prediction
- name: Python
  role: Training environment and runtime for the random-forest classifier and evaluation
    scripts (poetry-managed; scripts/train-test.py and notebooks)
- name: SigmaCCS
  role: Base GNN architecture that Mol2CCS extends; original high accuracy on CCSBase
    does not generalize, motivating confidence filtering
  repo: https://github.com/zmzhang/SigmaCCS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:3e08911d0655f8f961225c114519f20269ba98fee95d51beeeaaad64e45c4542
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enveda/ccs-prediction
  tools:
  - Mol2CCS
  - Python
  - SigmaCCS
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# confidence-model-training

## Summary

Trains a random-forest confidence model on Mol2CCS prediction residuals to identify CCS predictions likely to deviate more than 5% from the true value, then filters predictions by confidence score to lower MAE and MRE. This turns raw GNN CCS predictions into reliability-scored outputs before downstream use.

## When to use

Use when GNN-predicted CCS values (e.g. Mol2CCS outputs on METLIN-CCS or CCSBase) must be filtered for reliability, and per-prediction absolute relative error is available on a labeled subset to train a confidence classifier.

## When NOT to use

- Predictions already come with calibrated per-sample uncertainty (e.g. Monte Carlo dropout variance alone) and no labeled residuals exist to supervise a classifier.
- The task is plain CCS prediction accuracy rather than reliability filtering — use Mol2CCS training/splitting workflows instead.
- Data comes from a chemical space far outside METLIN-CCS or CCSBase; the paper shows model accuracy declines on structurally novel regions, so filtered scores may not transfer.

## Inputs

- Mol2CCS prediction outputs (predicted CCS values per molecule/adduct) for each training regime (train-METLIN-test-METLIN, train-CCSBase-test-CCSBase, cross-dataset)
- True experimental CCS values and SMILES/adduct records from METLIN-CCS (metlin.scripps.edu) or CCSBase v1.3 (ccsbase.net/query)
- Preprocessed .parquet inputs (e.g. metlin_train_3d.parquet, ccsbase_3d.parquet) with smiles, adduct, ccs and coordinates columns
- Binary error labels: positive where |predicted − true| / true ≤ 5%

## Outputs

- Trained random-forest confidence model scoring each prediction's reliability
- Precision–recall curve over classifier probability thresholds used to pick the operating point
- Filtered prediction subset (high-confidence predictions only) per training regime
- Recomputed MAE and MRE on the filtered subset vs unfiltered baseline, comparable to Fig. 5 and Supplementary Figs. 9–13

## How to apply

Label each Mol2CCS prediction as positive when its absolute relative error falls within 5% of the true CCS value, then train a random-forest classifier (scikit-learn) on these binary labels using the model's prediction features, sweeping the probability threshold via a precision-recall curve to pick an operating point for the high-confidence filter. Run the procedure separately for each training regime (train-METLIN-test-METLIN, train-CCSBase-test-CCSBase, cross-dataset), and optionally augment the confidence model's training set with in-domain samples — the paper shows MAE and MRE on the filtered subset decrease as in-domain sample size grows. Retain only predictions whose confidence score exceeds the chosen threshold and recompute MAE/MRE on the filtered subset; the confidence model may also be combined with Monte Carlo dropout to select the prediction with the highest confidence.

## Related tools

- **Mol2CCS** (GNN (extended SigmaCCS) whose CCS predictions are labeled, featureized and filtered by the confidence model) — https://github.com/enveda/ccs-prediction
- **Python** (Training environment and runtime for the random-forest classifier and evaluation scripts (poetry-managed; scripts/train-test.py and notebooks))
- **SigmaCCS** (Base GNN architecture that Mol2CCS extends; original high accuracy on CCSBase does not generalize, motivating confidence filtering) — https://github.com/zmzhang/SigmaCCS

## Examples

```
python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/ccsbase_3d.parquet" --dropout-rate 0.1 --epochs 400
```

## Evaluation signals

- For every training regime, MAE and MRE on the confidence-filtered subset must be lower than on the unfiltered predictions, matching the direction of improvements in Fig. 5 and Supplementary Figs. 9–13
- Positive label rate is reproducible: each prediction is labeled positive iff its absolute relative error is within 5% of the true CCS
- Precision–recall sweep shows the expected trade-off; the selected operating threshold is recorded and reused for filtering
- MAE/MRE improvement on the filtered subset increases as in-domain sample size added to the confidence model's training set grows
- Predictions removed by the filter concentrate in the high-|relative-error| tail rather than being sampled uniformly at random

## Limitations

- Evaluation is restricted mainly to small molecules and specific regions of chemical space (METLIN-CCS is based on synthetic structures), limiting transfer of the filter to other spaces.
- Variations in experimental conditions such as instrument types likely introduced inaccuracies in the CCS training values, which the confidence model only partially absorbs.
- The number of high-quality experimental CCS references is only in the low thousands, constraining how much in-domain data can augment the confidence model.
- Confidence filtering improves error metrics but does not restore the generalizability lost when training on one database and testing on another; the original high accuracy on CCSBase does not generalize to other chemical spaces.

## Evidence

- [discussion] Filtering improves MAE/MRE via random-forest confidence model: "we show that using confidence models to filter predictions can result in increased performance of the models."
- [results] Threshold and training steps from task card: [evidence span withheld — anchor status: not_found]
- [results] 5% absolute relative error cutoff: [evidence span withheld — anchor status: not_found]
- [discussion] Dropout combination with confidence model: "the confidence model that we trained could be used alongside Monte Carlo dropout to select the prediction with the highest confidence"
- [discussion] Generalizability decline motivating the filter: "the original high accuracy reported when the models were trained on CCSBase does not generalize to other chemical spaces"
