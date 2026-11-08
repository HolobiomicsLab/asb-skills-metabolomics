---
name: class-matched-versus-diverse-model-comparison
description: Use when evaluating whether a custom SVR CCS prediction training set
  (class-matched vs chemically diverse) sustains filtering power of annotation candidates
  as the mass-error window narrows from 10 ppm to isobaric 0 ppm.
when_to_use_negative:
- The training set is already class-matched to the analytes of interest AND only a
  single mass window (e.g. 10 ppm) is of interest — the cross-window comparison adds
  no information.
- Validation compounds lack experimentally measured CCS values, since true/false-positive
  classification requires the absolute relative ΔCCS of the true standard.
- Candidate lists have not yet been generated at the multiple mass windows (10–0 ppm);
  build them via PubChem/Entrez first, or the per-window AUC comparison cannot be
  run.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CCSP 2.0
  role: Generates the SVR CCS predictions for annotation candidates under both the
    chemically diverse and the class-matched (lipid superclass) training sets
  repo: https://github.com/facundof2016/CCSP2.0
- name: PubChem
  role: Source of candidate structures via the Identifier Exchange Service used to
    build the per-mass-window annotation candidate lists
- name: Mordred
  role: Calculates the 1613 2D molecular descriptors used as SVR features for CCS
    prediction of each candidate
- name: Python
  role: Environment for running the ΔCCS threshold sweep and scikit-learn roc_curve/roc_auc_score
    computations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:13552540143feb6428f55800f94d0712a3bac8c3ea8bd74b596550a19cd20997
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
  edam_operation: http://edamontology.org/operation_3927
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

# class-matched-versus-diverse-model-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks CCS-filtering performance (ROC AUC) of a class-matched SVR training set (lipid superclass) against a chemically diverse training set across PubChem candidate-list mass windows from 10 to 0 ppm, testing whether filtering efficacy is stable when the training set matches the analyte class.

## When to use

Use when evaluating whether a custom SVR CCS prediction training set (class-matched vs chemically diverse) sustains filtering power of annotation candidates as the mass-error window narrows from 10 ppm to isobaric 0 ppm.

## When NOT to use

- The training set is already class-matched to the analytes of interest AND only a single mass window (e.g. 10 ppm) is of interest — the cross-window comparison adds no information.
- Validation compounds lack experimentally measured CCS values, since true/false-positive classification requires the absolute relative ΔCCS of the true standard.
- Candidate lists have not yet been generated at the multiple mass windows (10–0 ppm); build them via PubChem/Entrez first, or the per-window AUC comparison cannot be run.

## Inputs

- Pre-built annotation candidate lists per mass window (10, 7, 5, 3, 1, 0 ppm) generated via PubChem/Entrez
- Experimentally measured CCS values for the 170-compound chemically diverse [M-H]- validation set
- Experimentally measured CCS values for the 33-compound lipid class-matched validation set
- CCSP-predicted CCS values for all candidates under both training-set conditions
- ΔCCS threshold sweep specification (absolute relative difference between predicted and measured CCS)

## Outputs

- results_table.csv with one row per mass window per data set (columns: mass window ppm, data set type, AUC)
- ROC curves of true-positive discovery rate versus false-positive discovery rate per mass window per data set
- Comparison of AUC trends: declining (diverse set) versus near-constant (lipid class-matched set)

## How to apply

Load the pre-built annotation candidate lists (PubChem/Entrez) built at 10, 7, 5, 3, 1 and 0 ppm mass windows for both the 170-compound chemically diverse [M-H]- validation set and the 33-compound lipid class-matched set, along with each compound's experimentally measured CCS and the CCSP-predicted CCS. For each mass window and each candidate, classify candidates as true or false positives across a sweep of ΔCCS thresholds, using the absolute relative difference between predicted and measured CCS of the true standard. Compute true-positive versus false-positive discovery rates across the ΔCCS threshold sweep with scikit-learn's roc_curve/roc_auc_score to obtain one AUC per mass window per data set. Tabulate both data sets side by side (diverse: 0.657, 0.633, 0.622, 0.616, 0.593, 0.589; lipid class-matched: 0.776, 0.754, 0.748, 0.768, 0.756, 0.756 for 10, 7, 5, 3, 1, 0 ppm) and export results_table.csv with mass window (ppm), data set type, and AUC columns. The rationale is that a training set matched to the analyte's molecular class (lipid superclass) should preserve near-constant filtering AUC as the mass window tightens to isobaric conditions, unlike the chemically diverse model whose AUC declines.

## Related tools

- **CCSP 2.0** (Generates the SVR CCS predictions for annotation candidates under both the chemically diverse and the class-matched (lipid superclass) training sets) — https://github.com/facundof2016/CCSP2.0
- **PubChem** (Source of candidate structures via the Identifier Exchange Service used to build the per-mass-window annotation candidate lists)
- **Mordred** (Calculates the 1613 2D molecular descriptors used as SVR features for CCS prediction of each candidate)
- **Python** (Environment for running the ΔCCS threshold sweep and scikit-learn roc_curve/roc_auc_score computations)

## Examples

```
from sklearn.metrics import roc_curve, roc_auc_score; fpr, tpr, _ = roc_curve(y_true, y_score); auc = roc_auc_score(y_true, y_score)
```

## Evaluation signals

- Diverse-set AUC values match 0.657, 0.633, 0.622, 0.616, 0.593, 0.589 for 10, 7, 5, 3, 1, 0 ppm within rounding tolerance (anchor checks: 10 ppm = 0.657; 0 ppm = 0.589).
- Lipid class-matched AUC values match 0.776, 0.754, 0.748, 0.768, 0.756, 0.756 for 10, 7, 5, 3, 1, 0 ppm within rounding tolerance (anchor checks: 10 ppm = 0.776; 0 ppm = 0.756).
- Qualitative pattern holds: diverse-set AUC declines monotonically-ish with narrower mass windows while lipid class-matched AUC stays near-constant (0.748–0.776 range).
- results_table.csv has exactly 12 rows (6 mass windows × 2 data set types) with columns mass window (ppm), data set type, and AUC.
- Each AUC falls between 0.5 and 1.0 and corresponds to a valid ROC curve of true-positive vs false-positive discovery rates across the ΔCCS threshold sweep.

## Limitations

- Class-matched training sets require prior knowledge of the analyte's molecular class (e.g. lipid superclass) and a curated class-specific training set; the specific training set must be defined before CCS prediction.
- The comparison is demonstrated for [M-H]- adducts only; CCSP 2.0 training sets do not support dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments.
- AUC gains from class-matching may not generalize to other molecular classes, adduct types, or IM-MS instrument platforms (drift tube vs travelling wave) not yet explored.

## Evidence

- [full_text] ROC AUC for class-matched lipid training set across mass windows: [evidence span withheld — anchor status: not_found]
- [full_text] Class-matched filtering remains effective across mass windows unlike diverse set: [evidence span withheld — anchor status: not_found]
- [readme] Repo availability for CCSP 2.0: [evidence span withheld — anchor status: external_doc]
