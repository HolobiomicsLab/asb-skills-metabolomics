---
name: logistic-regression-peak-filtering
description: Use when filtering MZmine 2 peak lists from nontargeted LC-MS metabolomics
  data and you need an interpretable, tabular-feature classifier that scores peak
  groups as true vs false without converting peaks to raster images.
when_to_use_negative:
- You need maximum filtering performance rather than interpretability — the image-based
  deep neural network retained 88% of true peaks and eliminated 89% of false peaks
  versus ~80%/~66% for this parameter model
- Input is raw LC-MS spectra rather than an MZmine 2 peak list — peak extraction and
  parameter computation must happen first
- The peak groups lack computed peak group parameters or the model requires peak-shape
  images rather than tabular features
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: R
  role: Custom R scripts implement the multiple logistic regression model scoring
    of peak group parameters
- name: MZmine 2
  role: Generates the non-filtered peak lists (more/less restrictive settings) from
    raw LC-MS data that supply the peak group parameters
- name: Random Forest
  role: Comparison model built from the original 59 peak group parameters that showed
    a similar level of performance
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:1a5b77129f51971aea0b7ec516c4845277dfc903f6d6e0969f8f270ce1d69b2f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MZmine 2
  - Multiple logistic regression
  - R
  - Random Forest
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# logistic-regression-peak-filtering

## Summary

A peak group parameter-based filtering skill that classifies LC-MS peak groups as true or false using a multiple logistic regression model built from 15 selected peak group parameters (e.g. medRelFWHM), implemented with custom R scripts as an interpretable alternative to the image-based deep neural network.

## When to use

Use when filtering MZmine 2 peak lists from nontargeted LC-MS metabolomics data and you need an interpretable, tabular-feature classifier that scores peak groups as true vs false without converting peaks to raster images.

## When NOT to use

- You need maximum filtering performance rather than interpretability — the image-based deep neural network retained 88% of true peaks and eliminated 89% of false peaks versus ~80%/~66% for this parameter model
- Input is raw LC-MS spectra rather than an MZmine 2 peak list — peak extraction and parameter computation must happen first
- The peak groups lack computed peak group parameters or the model requires peak-shape images rather than tabular features

## Inputs

- Non-filtered peak lists from MZmine 2 (v2.14) extracted with 'More Restrictive' or 'Less Restrictive' settings from raw LC-MS data
- Peak group parameter table (59 computed peak group parameters per peak group, e.g. medRelFWHM)
- Manually reviewed peak examples labeled true/false (e.g. 1304 training examples from Cohort 1) for model building and threshold selection
- Cohort 2 evaluation set of 3000 manually classified peak groups

## Outputs

- Probability score per peak group of being a true peak
- Filtered peak list retaining peaks above the selected score threshold
- Filter performance summary: fraction of true peaks retained and fraction of false peaks eliminated (e.g. ~80% retained / ~66% eliminated on Cohort 2)

## How to apply

Extract peak groups with MZmine 2 (more restrictive or less restrictive settings) and compute the peak group parameters for each candidate peak. Assess the predictive value of individual parameters with violin plots comparing true and false peaks and single-variable logistic regression, then build the multiple logistic regression model using a forward selection procedure where variables are added one at a time until model improvement stops; this yielded 15 peak group parameters in the final model, with medRelFWHM the most predictive of peak quality. Score each peak group with the model (custom R scripts), choose an optimum operating threshold from ROC analysis of manually labeled peaks, and compare true-peak retention and false-peak elimination against the conventional MZmine baseline. On Cohort 2 (3000 manually classified peak groups), this model retained ~80% of true peaks while eliminating ~66% of false peaks versus the deep neural network's 88%/89%, and a Random Forest built from the original 59 parameters showed a similar level of performance as a comparison model.

## Related tools

- **R** (Custom R scripts implement the multiple logistic regression model scoring of peak group parameters)
- **MZmine 2** (Generates the non-filtered peak lists (more/less restrictive settings) from raw LC-MS data that supply the peak group parameters)
- **Random Forest** (Comparison model built from the original 59 peak group parameters that showed a similar level of performance)

## Examples

```
Rscript score_peaks_logreg.R --input peak_group_parameters.csv --threshold 0.4725 --output filtered_peaks.csv
```

## Evaluation signals

- On the Cohort 2 evaluation set (3000 manually classified peak groups, more restrictive settings), the model should retain ~80% of manually labeled true peaks and eliminate ~66% of manually labeled false peaks
- ROC curve analysis of scores should show high specificity/sensitivity comparable to the paper's reported model performance; the DNN comparison reaches AUC 0.992/0.988 on training/test sets
- The forward selection procedure should stop at 15 peak group parameters in the final model, with medRelFWHM the most predictive
- Conventional more-restrictive MZmine baseline retains ~50% true / ~50% false peaks (11,026 features) — filtered results must exceed this
- Write per-model true-peak retention and false-peak removal rates to a summary table (dnn_param_filter_metrics.csv) and check the values against the paper's rates

## Limitations

- Underperforms the image-based deep neural network (~80% true retained / ~66% false eliminated vs 88%/89% on Cohort 2)
- Requires manually reviewed labeled peak examples (50% true / 50% false) for model building and threshold calibration
- Performance depends on MZmine 2 peak group parameters being computed identically to the original pipeline; only 15 of the 59 parameters entered the final model

## Evidence

- [results] the peak group parameter model retained 80% of true peaks while removing 66% of false peaks: [evidence span withheld — anchor status: not_found]
