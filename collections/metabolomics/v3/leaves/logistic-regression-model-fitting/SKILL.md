---
name: logistic-regression-model-fitting
description: Use when a labeled set of true/false LC-MS peaks has been scored with
  tabular peak group parameters (e.g., medRelFWHM) and you need an interpretable classifier
  or parameter ranking via logistic regression with forward selection.
when_to_use_negative:
- Input is raw LC-MS spectra not yet peak-picked — peak extraction with MZmine 2 and
  peak group parameter computation must precede this model.
- The goal is image-based peak classification rather than interpretable parameter-level
  modeling — the paper's deep neural network on rasterized peak shapes achieved higher
  AUC (0.988 test) and is a separate skill.
- Labeled true/false peak examples are unavailable — the forward-selection logistic
  regression requires manually reviewed training examples.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: MZmine 2
  role: Extracts raw LC-MS data into non-filtered peak lists using 'More Restrictive'
    and 'Less Restrictive' peak extraction parameters, providing the peak shape information
    from which the 59 peak group parameters are computed
- name: R
  role: Custom R scripts compute the 59 peak group parameters and fit the multiple
    logistic regression model via forward selection, plus violin plots and single-variable
    logistic regressions
- name: Random Forest
  role: Benchmark classifier developed from the original 59 peak group parameters
    showing similar performance to the logistic regression model
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:b3fc5c66c6ff5c49ff95eb18a53a18b12c04a29daab3a82b216900d764522898
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MZmine 2
  - R
  - Random Forest
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- linear-regression-model-fitting
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# logistic-regression-model-fitting

## Summary

Fits a multiple logistic regression classifier on LC-MS peak group shape parameters (59 candidate parameters derived from duration, height, area, FWHM, tailing factor, and asymmetry) to discriminate true from false spectral peaks, using forward variable selection until model improvement stops.

## When to use

Use when a labeled set of true/false LC-MS peaks has been scored with tabular peak group parameters (e.g., medRelFWHM) and you need an interpretable classifier or parameter ranking via logistic regression with forward selection.

## When NOT to use

- Input is raw LC-MS spectra not yet peak-picked — peak extraction with MZmine 2 and peak group parameter computation must precede this model.
- The goal is image-based peak classification rather than interpretable parameter-level modeling — the paper's deep neural network on rasterized peak shapes achieved higher AUC (0.988 test) and is a separate skill.
- Labeled true/false peak examples are unavailable — the forward-selection logistic regression requires manually reviewed training examples.

## Inputs

- Non-filtered MZmine 2 peak lists generated with 'More Restrictive' and 'Less Restrictive' peak extraction parameters
- Manually reviewed Cohort 1 peak examples: 1304 training, 740 calibration, 726 test (50% true / 50% false)
- Tabular matrix of 59 peak group shape parameters per peak group (computed in R from duration, height, area, FWHM, tailing factor, asymmetry)

## Outputs

- Multiple logistic regression model with 15 selected peak group parameters
- Per-peak true/false classification probability scores
- ROC curves and test-set AUC (>0.96 expected)
- Violin plots and single-variable logistic regression rankings identifying medRelFWHM as most predictive

## How to apply

Compute the 59 peak group parameters from six peak shape attributes (duration, height, area, FWHM, tailing factor, asymmetry) extracted by MZmine 2, then split manually reviewed Cohort 1 examples into 1304 training, 740 calibration, and 726 test examples. Fit the multiple logistic regression in R with a forward selection procedure, adding variables one at a time until model improvement stops, yielding 15 peak group parameters in the final model. Use violin plots of each parameter for true and false peaks plus single-variable logistic regression to rank predictive value; medRelFWHM (median peakFWHM/peak duration) emerged as most predictive. Evaluate by computing ROC curves on the held-out test set, requiring test AUC >0.96. A Random Forest trained on the same original 59 parameters can serve as a cross-check, showing similar performance.

## Related tools

- **MZmine 2** (Extracts raw LC-MS data into non-filtered peak lists using 'More Restrictive' and 'Less Restrictive' peak extraction parameters, providing the peak shape information from which the 59 peak group parameters are computed)
- **R** (Custom R scripts compute the 59 peak group parameters and fit the multiple logistic regression model via forward selection, plus violin plots and single-variable logistic regressions)
- **Random Forest** (Benchmark classifier developed from the original 59 peak group parameters showing similar performance to the logistic regression model)

## Examples

```
Rscript fit_peak_group_logreg.R --data 59_params.csv --labels manual_review.txt --method forward-selection --split 1304/740/726
```

## Evaluation signals

- Forward selection converges to 15 peak group parameters in the final multiple logistic regression model
- Test-set ROC AUC exceeds 0.96 on the 726 held-out examples
- medRelFWHM shows the greatest separation between true and false peak distributions in violin plots
- Model improvement stops when additional variables no longer help — adding further parameters beyond the 15 selected should not improve fit
- A Random Forest trained on the same 59 parameters should show a similar level of performance, serving as an algorithm-independent sanity check

## Limitations

- Forward selection is a greedy procedure — variables are added one at a time until model improvement stops, so globally optimal parameter subsets are not guaranteed.
- The model was trained on manually reviewed human plasma HILIC LC-MS peaks from Cohort 1 (N=78); transfer to other matrices or acquisition methods is not demonstrated here.
- The deep neural network on rasterized peak shapes outperformed this parameter-based model (test AUC 0.988), so the logistic regression is best used when interpretability or parameter ranking is required.
- Example-level ground truth depends on manual review of stacked peak plots built from the 63 samples in which each candidate peak was most intense.

## Evidence
