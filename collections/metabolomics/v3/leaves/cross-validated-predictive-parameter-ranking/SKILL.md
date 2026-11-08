---
name: cross-validated-predictive-parameter-ranking
description: Use when a peak table contains many candidate shape/quality features
  (e.g. duration, height, area, FWHM, tailing factor, asymmetry derivatives) and you
  must rank which features predict true vs false peaks before fitting a classifier.
when_to_use_negative:
- The goal is peak classification itself rather than parameter ranking — use the image-based
  deep neural network (score threshold 0.4725) instead, which reached AUC 0.988 on
  the test set
- Peaks have not been extracted and manually reviewed; unlabeled MZmine 2 peak lists
  cannot support supervised logistic regression or violin-plot separation
- Peak shape information is only available as per-sample raster images rather than
  computed peak group parameters
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3318
tools:
- name: R
  role: Custom R scripts implement the single-variable and multiple (forward-selected)
    logistic regression models and violin plots on the 59 peak group parameters
- name: MZmine 2
  role: Extracts non-filtered peak lists with both 'More Restrictive' and 'Less Restrictive'
    peak extraction parameters, providing the labeled peaks whose group parameters
    are ranked
- name: Random Forest
  role: Alternative model built from the original 59 peak group parameters, used as
    a comparison showing similar performance to the logistic regression
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:bf9f64aef60e281bce259f34765cd1f2e01127a5925fbc07a28b1e374e81ebed
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MZmine 2
  - R
  - Random Forest
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-validated-predictive-parameter-ranking

## Summary

Ranks engineered peak-quality features by their ability to discriminate true from false LC-MS spectral peaks, using violin-plot separation and single-variable logistic regression, then builds a multiple logistic regression model by forward selection on a train/calibration/test split. This identifies medRelFWHM (median peakFWHM/peak duration) as the most predictive of 59 peak group parameters.

## When to use

Use when a peak table contains many candidate shape/quality features (e.g. duration, height, area, FWHM, tailing factor, asymmetry derivatives) and you must rank which features best predict true vs false peaks before fitting a classifier.

## When NOT to use

- The goal is peak classification itself rather than parameter ranking — use the image-based deep neural network (score threshold 0.4725) instead, which reached AUC 0.988 on the test set
- Peaks have not been extracted and manually reviewed; unlabeled MZmine 2 peak lists cannot support supervised logistic regression or violin-plot separation
- Peak shape information is only available as per-sample raster images rather than computed peak group parameters

## Inputs

- Manually reviewed (labeled true/false) peak examples from MZmine 2 non-filtered peak lists (Cohort 1, N = 78 human plasma HILIC LC-MS samples; 1304 train / 740 calibration / 726 test)
- 59 peak group shape parameters derived from six peak attributes (duration, height, area, FWHM, tailing factor, asymmetry)

## Outputs

- Ranked list of peak group parameters by predictive value (medRelFWHM ranked first)
- Violin plots of each parameter for true vs false peaks
- Single-variable logistic regression fits per parameter
- Forward-selected multiple logistic regression model with 15 peak group parameters and test-set ROC/AUC (>0.96)

## How to apply

Compute 59 peak group shape parameters from six peak attributes (duration, height, area, FWHM, tailing factor, asymmetry) on manually reviewed MZmine 2 peaks, then split the examples into 1304 training, 740 calibration, and 726 test cases with a 50% true / 50% false balance in training. For each parameter, draw violin plots of its distribution for true vs false peaks and fit single-variable logistic regression models in R to quantify discriminative value; in this study medRelFWHM (median peakFWHM/peak duration) showed the greatest separation. Then train a multiple logistic regression via forward selection, adding variables one at a time until model improvement stops, yielding 15 final parameters. Validate by computing ROC curves on the held-out test set, where the model achieved AUC > 0.96.

## Related tools

- **R** (Custom R scripts implement the single-variable and multiple (forward-selected) logistic regression models and violin plots on the 59 peak group parameters)
- **MZmine 2** (Extracts non-filtered peak lists with both 'More Restrictive' and 'Less Restrictive' peak extraction parameters, providing the labeled peaks whose group parameters are ranked)
- **Random Forest** (Alternative model built from the original 59 peak group parameters, used as a comparison showing similar performance to the logistic regression)

## Examples

```
Rscript -e 'm <- glm(true_peak ~ medRelFWHM, family=binomial, data=train); p <- predict(m, newdata=test, type="response"); roc(test$true_peak, p)'
```

## Evaluation signals

- medRelFWHM must show the greatest separation between true and false peak distributions in violin plots and the strongest single-variable logistic regression performance
- The forward-selected multiple logistic regression must use 15 peak group parameters as final variables
- Test-set ROC AUC must exceed 0.96 (article reports >0.96; the neural net comparison reached 0.988)
- Training set must be balanced at 50% true peaks and 50% false peaks, with 740 calibration and 726 test examples held out
- A Random Forest trained on the same 59 parameters should show a similar level of performance, serving as a cross-check

## Limitations

- Only 15 of the 59 parameters enter the final model; individual parameter rankings beyond medRelFWHM depend on the specific human plasma HILIC LC-MS cohorts used
- Requires substantial manual review effort (2770 labeled peak examples across train/calibration/test), limiting scalability compared to the image-based DNN
- The forward selection stops 'until model improvement stops', so the final variable count is data-dependent and may differ on other datasets

## Evidence

- [results] Violin plots of each peak group parameter for true and false peaks as well as single variable logistic regression models were used to assess the predictive value: [evidence span withheld — anchor status: not_found]
