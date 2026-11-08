---
name: forward-variable-selection
description: Use when you have manually labeled true/false peak examples and many
  candidate shape parameters (e.g., 59 peak group parameters from duration, height,
  area, FWHM, tailing factor, asymmetry) and need a small interpretable logistic regression
  model.
when_to_use_negative:
- When the goal is maximum classification performance on peak images rather than an
  interpretable parameter-based model — the image-based deep neural network reached
  AUC 0.992/0.988 and outperformed this approach.
- When peak examples have not been manually reviewed/labeled — forward selection requires
  labeled true/false training examples.
- When the number of candidate parameters is small enough for exhaustive evaluation;
  forward selection is a greedy heuristic and may miss optimal variable subsets.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: R
  role: Environment used with custom R scripts to implement the forward selection
    logistic regression and ROC evaluation
- name: MZmine 2
  role: Extracts raw data into non-filtered peak lists from which the peak group parameters
    fed into the regression are computed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:7fa97de9fae8b1a73397deb17fbf7389b764f60c296d8fd04410dd315d815501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MZmine 2
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# forward-variable-selection

## Summary

Builds a multiple logistic regression peak-quality classifier by greedily adding peak group shape parameters one at a time until model improvement stops, yielding a compact parsimonious model (e.g., 15 of 59 parameters with medRelFWHM as the top predictor) for separating true from false LC-MS spectral peaks.

## When to use

Use when you have manually labeled true/false peak examples and many candidate shape parameters (e.g., 59 peak group parameters from duration, height, area, FWHM, tailing factor, asymmetry) and need a small interpretable logistic regression model.

## When NOT to use

- When the goal is maximum classification performance on peak images rather than an interpretable parameter-based model — the image-based deep neural network reached AUC 0.992/0.988 and outperformed this approach.
- When peak examples have not been manually reviewed/labeled — forward selection requires labeled true/false training examples.
- When the number of candidate parameters is small enough for exhaustive evaluation; forward selection is a greedy heuristic and may miss optimal variable subsets.

## Inputs

- Manually reviewed peak examples (training/calibration/test splits) with computed peak group shape parameters
- 59 peak group parameters per peak derived from six peak attributes: duration, height, area, FWHM, tailing factor, asymmetry
- Binary labels: true peak vs. false peak

## Outputs

- Multiple logistic regression model with 15 selected peak group parameters
- Ranked variable importance (medRelFWHM identified as most predictive)
- ROC curves and test-set AUC (expected > 0.96)
- Violin plots and single-variable logistic regression performance per parameter

## How to apply

Compute the candidate parameter set from manually reviewed peak examples — here 59 peak group parameters derived from six peak attributes (duration, height, area, FWHM, tailing factor, asymmetry) of LC-MS peaks. Split the reviewed examples into training, calibration, and test sets (e.g., 1304 training, 740 calibration, 726 test from Cohort 1, balanced 50% true / 50% false). Run a forward selection procedure: start with the single most predictive variable (medRelFWHM, the median peakFWHM/peak duration), add variables one at a time, and stop when model improvement ceases, yielding 15 final variables. Evaluate the resulting multiple logistic regression model by ROC curves on the held-out test set, requiring test AUC > 0.96; use violin plots and single-variable logistic regression models to confirm which parameters (medRelFWHM) best separate true and false peak distributions.

## Related tools

- **R** (Environment used with custom R scripts to implement the forward selection logistic regression and ROC evaluation)
- **MZmine 2** (Extracts raw data into non-filtered peak lists from which the peak group parameters fed into the regression are computed)

## Evaluation signals

- Final model uses 15 peak group parameters selected by forward selection from the original 59
- Test-set AUC of the multiple logistic regression model exceeds 0.96
- medRelFWHM shows the greatest separation between true and false peak distributions in violin plots and is the top single-variable predictor
- Training/calibration/test splits are 1304/740/726 manually reviewed peak examples (50% true, 50% false in training)
- Similar-performance sanity check: a Random Forest trained on the same 59 parameters should show a comparable level of performance

## Limitations

- The parameter-based logistic regression model discriminates less well than the image-based deep neural network (test AUC > 0.96 vs. 0.988).
- Forward selection is greedy: it adds variables one at a time and stops when improvement stops, so it may not find the globally optimal parameter subset.
- Model was trained on manually reviewed human plasma HILIC LC-MS peaks from Cohort 1; transfer to other data types is not demonstrated by this procedure alone.

## Evidence
