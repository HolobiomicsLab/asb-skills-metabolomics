---
name: cohort-evaluation-reproducibility-check
description: Use when a trained LC-MS peak-classification model must be evaluated
  on a held-out, manually classified cohort to verify that reported true-peak retention
  and false-peak elimination rates are reproducible.
when_to_use_negative:
- Input is already a fully filtered, curated feature table with no manually labeled
  true/false peak groups — there is no ground truth against which retention/elimination
  rates can be computed.
- Peak groups were extracted with parameters different from the paper's 'More Restrictive'
  or 'Less Restrictive' MZmine 2 settings, so the reported baseline (11,026 features,
  ~50/50 true/false split) does not apply.
- The goal is model training rather than independent-cohort evaluation — for training
  use the Cohort 1 examples (1304 training / 740 calibration / 726 test) via EDML_deep_learning2.py
  instead.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine 2
  role: Source of the peak lists under 'More Restrictive' and 'Less Restrictive' extraction
    settings that define the conventional baseline being compared against
- name: R
  role: Environment for scoring peak groups with the 15-parameter multiple logistic
    regression model and computing retention/elimination rates
- name: EDML_deep_learning2.py
  role: Script used to train the image-based deep neural network model that is then
    applied to the Cohort 2 evaluation set
  repo: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
- name: Deep neural network (image based)
  role: Primary classifier applied to rasterized peak images with score threshold
    0.4725 to classify true vs false peaks
- name: Multiple logistic regression
  role: Comparator model built from 15 peak group parameters via forward selection,
    scored on the same Cohort 2 peak groups
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:17becbb9ac14a0e3529c60dfa6214d390d019cbb00e8ed37152f484e7ff22363
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
  tools:
  - Deep neural network (image based)
  - EDML_deep_learning2.py
  - MZmine 2
  - Multiple logistic regression
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cohort-evaluation-reproducibility-check

## Summary

Reproduce the paper's independent-cohort evaluation by scoring manually classified peak groups (e.g. the 3000 Cohort 2 groups) with the image-based DNN at score threshold 0.4725 and the 15-parameter logistic regression model, then compare true-peak retention and false-peak elimination against the conventional more-restrictive MZmine 2 baseline.

## When to use

Use when a trained LC-MS peak-classification model must be evaluated on a held-out, manually classified cohort to verify that reported true-peak retention and false-peak elimination rates are reproducible.

## When NOT to use

- Input is already a fully filtered, curated feature table with no manually labeled true/false peak groups — there is no ground truth against which retention/elimination rates can be computed.
- Peak groups were extracted with parameters different from the paper's 'More Restrictive' or 'Less Restrictive' MZmine 2 settings, so the reported baseline (11,026 features, ~50/50 true/false split) does not apply.
- The goal is model training rather than independent-cohort evaluation — for training use the Cohort 1 examples (1304 training / 740 calibration / 726 test) via EDML_deep_learning2.py instead.

## Inputs

- Manually classified peak groups from Cohort 2 (N=526, 3000 groups) extracted with more restrictive MZmine 2 settings, downloaded from doi:10.25345/C5FD2F
- Raw nontargeted HILIC LC-MS peak shape information (peak information from the 63 samples in which each candidate peak was most intense)
- Trained image-based deep neural network model and 15-parameter multiple logistic regression model
- Manually labeled true/false peak classifications used as ground truth

## Outputs

- Summary table (dnn_param_filter_metrics.csv) of per-model true-peak retention rate and false-peak elimination rate
- Comparison against the conventional more-restrictive baseline (11,026 features, ~50% true / ~50% false)
- Optional ROC curves / score-threshold diagnostics for the DNN at cutoff 0.4725

## How to apply

Download the evaluation deposit (doi:10.25345/C5FD2F) containing the 3000 manually classified peak groups from Cohort 2 extracted with the more restrictive MZmine 2 settings. Generate raster images for each peak group using the peak information from the 63 samples in which the candidate peak was most intense (rows = samples, columns = scan points, pixel = intensity), then score each group with the image-based deep neural network and apply the optimum score threshold of 0.4725. In parallel, score the same groups with the 15-parameter multiple logistic regression model built by forward selection. Compute the fraction of manually labeled true peaks retained and the fraction of manually labeled false peaks eliminated for each model, and compare both against the conventional more-restrictive baseline (~50% true retained, ~50% false retained, 11,026 features). Validation: the DNN should retain ~88% of true peaks and eliminate ~89% of false peaks; the parameter model should retain ~80% of true peaks and eliminate ~66% of false peaks; the DNN under less restrictive settings should retain ~98% of true peaks and reduce false peaks by more than 50%.

## Related tools

- **MZmine 2** (Source of the peak lists under 'More Restrictive' and 'Less Restrictive' extraction settings that define the conventional baseline being compared against)
- **R** (Environment for scoring peak groups with the 15-parameter multiple logistic regression model and computing retention/elimination rates)
- **EDML_deep_learning2.py** (Script used to train the image-based deep neural network model that is then applied to the Cohort 2 evaluation set) — https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
- **Deep neural network (image based)** (Primary classifier applied to rasterized peak images with score threshold 0.4725 to classify true vs false peaks)
- **Multiple logistic regression** (Comparator model built from 15 peak group parameters via forward selection, scored on the same Cohort 2 peak groups)

## Examples

```
python EDML_deep_learning2.py  # after downloading Cohort 2 evaluation data from doi:10.25345/C5FD2F, then apply threshold 0.4725 and export dnn_param_filter_metrics.csv
```

## Evaluation signals

- DNN on Cohort 2 more-restrictive set must retain ~88% of true peaks and eliminate ~89% of false peaks at score threshold 0.4725.
- 15-parameter logistic regression model must retain ~80% of true peaks and eliminate ~66% of false peaks on the same set.
- Conventional more-restrictive baseline must reproduce 11,026 features with ~50% true and ~50% false positive peaks.
- Under less restrictive settings, the DNN must capture ~98% of true positive peaks and reduce false peak detection by more than 50%.
- Training/test AUCs of 0.992 and 0.988 must be reproduced when the DNN is retrained from scratch via EDML_deep_learning2.py.

## Limitations

- Evaluation depends on manually classified peak groups; results are only valid for the 3000 Cohort 2 groups and the specific more/less restrictive MZmine 2 parameter sets used.
- The score threshold 0.4725 was selected from calibration data (Supporting Information selection criteria); other thresholds will shift the true/false rates.
- The DNN requires peak shape information from the 63 samples where the peak was most intense — peak groups with fewer eligible samples or different imaging conventions will not be scored comparably.
- Random Forest on the original 59 peak group parameters showed similar performance but is not the primary model reported for this evaluation.

## Evidence

- [results] more restrictive baseline definition: "Under more restrictive settings conventional peak selection resulted in approximately 50% true positive and 50% false positive peaks, capturing a total of 11,026 features"
