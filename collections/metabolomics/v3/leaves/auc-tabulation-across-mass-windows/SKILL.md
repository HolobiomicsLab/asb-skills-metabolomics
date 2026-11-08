---
name: auc-tabulation-across-mass-windows
description: Use when you need to test whether a threshold-based candidate filtering
  metric (e.g., ΔCCS between predicted and measured CCS) remains effective as candidate-list
  mass tolerance tightens from 10 ppm down to the isobaric 0 ppm case.
when_to_use_negative:
- Input is raw IM-MS spectra rather than pre-built candidate lists with known-standard
  CCS measurements — this skill evaluates filtering, not annotation generation
- The candidate set lacks ground-truth true/false-positive labels tied to experimentally
  measured CCS standards, since AUC cannot be computed without them
- You only need a single fixed mass window (e.g., the ΔCCS 2.8% / 10 ppm filtering
  scenario) rather than a cross-window stability comparison
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CCSP 2.0
  role: Supplies the SVR-based CCS predictions for each annotation candidate that
    are compared against measured CCS to define the ΔCCS filtering metric
  repo: https://github.com/facundof2016/CCSP2.0
- name: Python
  role: Runs the ROC/AUC computation (scikit-learn roc_curve/roc_auc_score) and writes
    the results table
- name: PubChem
  role: Source of candidate-list generation (Identifier Exchange Service / Entrez)
    at each mass window
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:041ab7e2d364214b4ce7c592c2998a46cb804547f2d930fc216098af41c6da3a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Mordred
  - PubChem
  - Python
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# auc-tabulation-across-mass-windows

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes and tabulates ROC AUC scores for CCS-based filtering of metabolite annotation candidates across a sweep of candidate-list mass windows (10, 7, 5, 3, 1, 0 ppm), enabling comparison of filtering efficacy between chemically diverse and class-matched (lipid superclass) CCSP 2.0 training sets.

## When to use

Use when you need to test whether a threshold-based candidate filtering metric (e.g., ΔCCS between predicted and measured CCS) remains effective as candidate-list mass tolerance tightens from 10 ppm down to the isobaric 0 ppm case.

## When NOT to use

- Input is raw IM-MS spectra rather than pre-built candidate lists with known-standard CCS measurements — this skill evaluates filtering, not annotation generation
- The candidate set lacks ground-truth true/false-positive labels tied to experimentally measured CCS standards, since AUC cannot be computed without them
- You only need a single fixed mass window (e.g., the ΔCCS 2.8% / 10 ppm filtering scenario) rather than a cross-window stability comparison

## Inputs

- Pre-built annotation candidate lists per mass window (10, 7, 5, 3, 1, 0 ppm) generated via PubChem/Entrez for the 170-compound chemically diverse [M-H]- validation set
- Pre-built annotation candidate lists per mass window for the 33-compound lipid class-matched set
- Experimentally measured CCS values for each true standard compound
- CCSP 2.0-predicted CCS values for each annotation candidate
- True-positive/ground-truth labels linking each candidate to its known standard

## Outputs

- results_table.csv with one row per mass window per data set and columns: mass window (ppm), data set type, AUC
- Expected AUC values — diverse set: 0.657, 0.633, 0.622, 0.616, 0.593, 0.589 (10, 7, 5, 3, 1, 0 ppm); lipid class-matched set: 0.776, 0.754, 0.748, 0.768, 0.756, 0.756
- Per-window ROC curves (true-positive vs false-positive discovery rates across the ΔCCS threshold sweep)

## How to apply

For each mass window (10, 7, 5, 3, 1, and 0 ppm), load the pre-built annotation candidate lists and, for each candidate, label it true or false positive by comparing its predicted CCSP CCS to the experimentally measured CCS of the true standard. Sweep ΔCCS thresholds (absolute relative percent difference between predicted and measured CCS of the true standard) and compute true-positive versus false-positive discovery rates with scikit-learn's roc_curve/roc_auc_score to obtain one AUC per mass window per training-set condition. Tabulate results into a single comparison table (one row per mass window per data set, with columns for mass window in ppm, data set type, and AUC). The rationale, per Rainey et al. 2022, is that AUC stability across narrowing windows distinguishes training-set choice effects: the chemically diverse 170-compound [M-H]- set declines with narrower windows while the 33-compound lipid class-matched set stays near-constant. Validate by matching known endpoint AUCs (diverse: 0.657 at 10 ppm, 0.589 at 0 ppm; lipid: 0.776 at 10 ppm, 0.756 at 0 ppm) within rounding tolerance.

## Related tools

- **CCSP 2.0** (Supplies the SVR-based CCS predictions for each annotation candidate that are compared against measured CCS to define the ΔCCS filtering metric) — https://github.com/facundof2016/CCSP2.0
- **Python** (Runs the ROC/AUC computation (scikit-learn roc_curve/roc_auc_score) and writes the results table)
- **PubChem** (Source of candidate-list generation (Identifier Exchange Service / Entrez) at each mass window)

## Examples

```
from sklearn.metrics import roc_curve, roc_auc_score; fpr, tpr, _ = roc_curve(labels, -delta_ccs); auc = roc_auc_score(labels, -delta_ccs)
```

## Evaluation signals

- Diverse-set AUC endpoints match: 10 ppm = 0.657 and 0 ppm = 0.589 within rounding tolerance
- Lipid class-matched AUC endpoints match: 10 ppm = 0.776 and 0 ppm = 0.756 within rounding tolerance
- Qualitative pattern reproduced: monotonically declining AUC with narrower mass windows for the chemically diverse set, but near-constant AUC (0.748–0.776) across all windows for the lipid class-matched set
- results_table.csv has exactly 12 rows (6 mass windows × 2 data sets) with the required columns: mass window (ppm), data set type, AUC
- Each AUC is bounded in (0, 1] and each 0 ppm (isobaric) window uses candidate lists where all candidates are isobaric with the measured precursor

## Limitations

- Results are specific to the training-set conditions tested (170-compound chemically diverse [M-H]- set vs 33-compound lipid class-matched set); a user must define the specific training set prior to CCS prediction
- CCSP 2.0's current training sets do not support dimers such as [2M+H]+ or [2M-H]-, other less common adducts, or in-source fragments found in ESI-MS experiments
- Running CCSP 2.0 requires familiarity with Python (scikit-learn V1.0.2 or later required), which may limit reproducibility of the underlying predictions being evaluated

## Evidence

- [abstract] CCS predictions via CCSP allowed filtering of 36.1% of incorrect structures while retaining a total of 100% of the correct annotations: "CCS predictions via CCSP allowed filtering of 36.1% of incorrect structures while retaining a total of 100% of the correct annotations"
- [readme] The current version of CCSP 2.0 requires Sklearn V1.0.2 or later. A full list of package requirements is available in the requirements.txt file.: [evidence span withheld — anchor status: external_doc]
