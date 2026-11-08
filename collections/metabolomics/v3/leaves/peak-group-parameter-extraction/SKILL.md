---
name: peak-group-parameter-extraction
description: Use when MZmine 2 non-filtered peak lists from LC-MS metabolomics data
  must be converted into numeric peak shape features (e.g., medRelFWHM) for training
  or applying true/false peak classifiers such as logistic regression or Random Forest.
when_to_use_negative:
- The goal is image-based deep neural network peak classification, which uses rasterized
  peak shape images rather than the 59 numeric peak group parameters
- Peaks have already been classified and filtered — this skill produces features,
  not final true/false labels
- Data was not processed through MZmine 2 peak detection, so peak attributes like
  FWHM, tailing factor, and asymmetry are unavailable
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine 2
  role: Extracts raw LC-MS data into non-filtered peak lists with the peak attributes
    (duration, height, area, FWHM, tailing factor, asymmetry) from which the 59 peak
    group parameters are computed
- name: R
  role: Custom R scripts used to compute the peak group parameters, fit the single-variable
    and forward-selection multiple logistic regression models, and produce violin
    plots
  repo: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
- name: Random Forest
  role: Alternative classifier trained on the original 59 peak group parameters to
    validate feature quality
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:982a3e6bd910c7f7cff0ebe8081aa5dd9e0b01452b5a4c62b19622008ae6f716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
  tools:
  - MZmine 2
  - R
  - Random Forest
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# peak-group-parameter-extraction

## Summary

Computes 59 peak group shape parameters from six peak attributes (duration, height, area, FWHM, tailing factor, asymmetry) extracted from MZmine 2 peak lists of nontargeted LC-MS data, to serve as engineered features for peak quality classification (e.g., multiple logistic regression where medRelFWHM was most predictive).

## When to use

Use when MZmine 2 non-filtered peak lists from LC-MS metabolomics data must be converted into numeric peak shape features (e.g., medRelFWHM) for training or applying true/false peak classifiers such as logistic regression or Random Forest.

## When NOT to use

- The goal is image-based deep neural network peak classification, which uses rasterized peak shape images rather than the 59 numeric peak group parameters
- Peaks have already been classified and filtered — this skill produces features, not final true/false labels
- Data was not processed through MZmine 2 peak detection, so peak attributes like FWHM, tailing factor, and asymmetry are unavailable

## Inputs

- Non-filtered MZmine 2 peak lists generated with both 'More Restrictive' and 'Less Restrictive' peak extraction parameters
- Raw nontargeted HILIC LC-MS metabolomics data (e.g., human plasma cohorts Cohort 1 N=78 and Cohort 2 N=526)
- Peak information from the 63 samples in which each candidate peak was most intense

## Outputs

- Table of 59 peak group shape parameters per candidate peak group (including medRelFWHM = median peakFWHM / peak duration)
- Violin plots of each peak group parameter for true vs false peaks
- Feature matrix for multiple logistic regression (15 selected variables) or Random Forest (59 parameters) peak quality classifiers

## How to apply

Extract raw LC-MS data with MZmine 2 using both 'More Restrictive' and 'Less Restrictive' peak extraction parameters to generate non-filtered peak lists. For each candidate feature, generate stacked peak plots using peak information from the 63 samples in which the candidate peak was most intense, then compute the 59 peak group parameters derived from the six peak attributes: duration, height, area, FWHM, tailing factor, and asymmetry. The key derived parameter is medRelFWHM, the median peakFWHM divided by peak duration, which the paper identified as the single most predictive parameter of peak quality via violin plots and single-variable logistic regression. Feed the parameter table into downstream models: a 15-variable multiple logistic regression built by forward selection (test AUC > 0.96) or a Random Forest trained on all 59 parameters, which showed similar performance. Split manually reviewed examples (e.g., 1304 training / 740 calibration / 726 test) before modeling to preserve an unbiased test set.

## Related tools

- **MZmine 2** (Extracts raw LC-MS data into non-filtered peak lists with the peak attributes (duration, height, area, FWHM, tailing factor, asymmetry) from which the 59 peak group parameters are computed)
- **R** (Custom R scripts used to compute the peak group parameters, fit the single-variable and forward-selection multiple logistic regression models, and produce violin plots) — https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
- **Random Forest** (Alternative classifier trained on the original 59 peak group parameters to validate feature quality)

## Examples

```
Rscript peak_group_parameters.R --peaklist more_restrictive.csv --output 59_params.csv
```

## Evaluation signals

- medRelFWHM shows the greatest separation between true and false peak distributions in violin plots and the highest single-variable logistic regression predictive value
- A multiple logistic regression model built via forward selection on the 59 parameters reaches test-set AUC > 0.96 (paper achieved 15 final variables)
- Peak extraction recapitulates the expected class balance, e.g., ~50% true / 50% false under more restrictive settings (11,026 features) and ~75% false positives under less restrictive settings
- A Random Forest trained on the same 59 parameters shows similar performance to the logistic regression model, confirming the features carry consistent signal

## Limitations

- Feature computation depends on MZmine 2 peak detection quality; parameter definitions are tied to MZmine's peak attributes and peak extraction restrictiveness settings
- Training labels require manual review of peak examples (stacked peak plots across the 63 most intense samples), which does not scale automatically
- The article does not name a specific deep learning framework or hosted code version; the README points only to dataset download (https://doi.org/doi:10.25345/C5FD2F) and the EDML_deep_learning2.py script

## Evidence

- [results] Non-filtered peak lists were then generated using MZmine 2 where the raw data was extracted using both "More Restrictive" and "Less Restrictive" peak extraction parameters: [evidence span withheld — anchor status: not_found]
