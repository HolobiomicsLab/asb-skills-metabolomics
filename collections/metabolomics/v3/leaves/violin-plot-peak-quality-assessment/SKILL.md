---
name: violin-plot-peak-quality-assessment
description: Use when you have manually reviewed LC-MS peak examples labeled true/false
  and need to identify which of the 59 peak group parameters separates true from false
  peaks before building a classifier.
when_to_use_negative:
- You need a full multivariate classifier — this skill only assesses individual parameters;
  use the forward-selected multiple logistic regression model (15 variables) or the
  image-based deep neural network for classification instead
- Peak examples are not manually reviewed/labeled true vs. false — distributional
  separation cannot be evaluated without ground-truth labels
- Your features are raw raster peak images rather than computed scalar shape parameters
  — the image-based deep neural network workflow applies in that case
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3318
tools:
- name: R
  role: Custom R scripts used to compute peak group parameters and produce violin
    plots and single-variable logistic regression models
- name: MZmine 2
  role: Extracts raw data into non-filtered peak lists under 'More Restrictive' and
    'Less Restrictive' settings, providing the candidate peaks whose parameters are
    assessed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:49082b84023684bc3a3fc18a2c2cd05e76bcada09a73bb83c959e246746e3ae6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MZmine 2
  - R
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# violin-plot-peak-quality-assessment

## Summary

Assesses the predictive value of individual LC-MS peak group shape parameters (e.g., medRelFWHM, derived from duration, height, area, FWHM, tailing factor, asymmetry) by plotting violin plots of each parameter's distribution for manually labeled true vs. false peaks and complementing them with single-variable logistic regression models.

## When to use

Use when you have manually reviewed LC-MS peak examples labeled true/false and need to identify which of the 59 peak group parameters best separates true from false peaks before building a classifier.

## When NOT to use

- You need a full multivariate classifier — this skill only assesses individual parameters; use the forward-selected multiple logistic regression model (15 variables) or the image-based deep neural network for classification instead
- Peak examples are not manually reviewed/labeled true vs. false — distributional separation cannot be evaluated without ground-truth labels
- Your features are raw raster peak images rather than computed scalar shape parameters — the image-based deep neural network workflow applies in that case

## Inputs

- Manually reviewed LC-MS peak examples labeled true or false (e.g., 1304 training, 740 calibration, 726 test peak examples from Cohort 1 human plasma)
- Peak group shape parameters computed from six peak attributes (duration, height, area, FWHM, tailing factor, asymmetry) — 59 parameters total, including medRelFWHM
- Stacked peak plots generated from the 63 samples in which each candidate peak was most intense (used for manual review)

## Outputs

- Violin plots of each peak group parameter's distribution for true vs. false peaks
- Single-variable logistic regression results quantifying each parameter's individual predictive value
- Ranked assessment identifying the most predictive parameter (medRelFWHM) for use in peak quality filtering

## How to apply

Compute the 59 peak group shape parameters from six peak attributes (duration, height, area, FWHM, tailing factor, asymmetry) for each manually reviewed peak example (e.g., 1304 training, 740 calibration, 726 test examples from Cohort 1). For each peak group parameter, generate a violin plot showing the distribution of the parameter for true peaks versus false peaks, and fit a single-variable logistic regression model to quantify its standalone predictive power. Compare separation between the true and false peak distributions across parameters: in this paper, medRelFWHM (median peakFWHM/peak duration) showed the greatest separation and was the most predictive single parameter of peak quality. Use this ranking to guide forward selection of 15 parameters into the final multiple logistic regression peak-filtering model, and to justify inclusion of the top parameter in downstream quality filtering.

## Related tools

- **R** (Custom R scripts used to compute peak group parameters and produce violin plots and single-variable logistic regression models)
- **MZmine 2** (Extracts raw data into non-filtered peak lists under 'More Restrictive' and 'Less Restrictive' settings, providing the candidate peaks whose parameters are assessed)

## Evaluation signals

- The violin plot for the top parameter (medRelFWHM) shows visibly greater separation between true and false peak distributions than any other peak group parameter
- Single-variable logistic regression identifies medRelFWHM as the most predictive individual parameter of peak quality
- The number of parameters assessed matches the 59 peak group parameters built from the six peak shape attributes
- Ranked parameter assessments are consistent with the final forward-selected multiple logistic regression model containing 15 peak group parameters

## Limitations

- Assessment is limited to the 59 peak group parameters derived from six peak shape attributes; it does not capture full peak shape information (which the image-based deep neural network exploits via raster images)
- Results depend on manual review labeling of peak examples (Cohort 1: 1304 training, 740 calibration, 726 test examples); biases in manual review propagate to parameter rankings
- Single-parameter assessment ignores interactions between parameters, which are handled by the multiple logistic regression (forward selection) and Random Forest models

## Evidence

- [results] Violin plots of each peak group parameter for true and false peaks as well as single variable logistic regression models were used to assess the predictive value: [evidence span withheld — anchor status: not_found]
