---
name: metabolite-panel-feature-selection
description: Use when a supervised classifier built on the full preprocessed GCxGC-TOFMS
  metabolite matrix separates known clinical classes (e.
when_to_use_negative:
- The analysis goal is unsupervised exploration of group structure — use the PCA scores
  approach (as in S1 Fig) before any supervised panel selection.
- The goal is separating S. Typhi carriage from S. Paratyphi A carriage — that requires
  the multi-class OPLS-DA model along the second component (p = 0.0031), not the combined-carriage
  two-class panel.
- The input matrix is not preprocessed GCxGC-TOFMS plasma data or the class labels
  for carriage vs non-carriage are missing, so supervised OPLS-DA fitting and cross-validated
  ROC scoring cannot be performed.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: GCxGC-TOFMS
  role: Source instrument producing the 195-metabolite plasma feature matrix from
    which the five-metabolite panel is subset.
- name: SIMCA
  role: Used (version 14, Umetrics) to fit the two-class OPLS-DA panel model with
    latent significance covariance loadings and seven-fold cross-validation.
- name: MATLAB
  role: Used (R2014b) to construct the ROC curve from cross-validated OPLS-DA scores,
    compute the AUC, and bootstrap 1,000 percentile resamples for 95% CI.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:544d9b9487035fb7bb315a1c6ee0d77da41b6d8beacf9952592e7c40cc876ae6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  - MATLAB
  - SIMCA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# metabolite-panel-feature-selection

## Summary

Selects a minimal co-varying metabolite panel from a GCxGC-TOFMS metabolite matrix and validates it as a combined latent diagnostic pattern, using OPLS-DA rather than individual-metabolite testing. In the typhoid carriage study this yielded a five-metabolite panel (glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399) with AUC 0.935.

## When to use

Use when a supervised classifier built on the full preprocessed GCxGC-TOFMS metabolite matrix separates known clinical classes (e.g. Salmonella carriage vs non-carriage controls) and you must reduce 195 metabolites to a minimal panel whose combined diagnostic performance exceeds any single metabolite.

## When NOT to use

- The analysis goal is unsupervised exploration of group structure — use the PCA scores approach (as in S1 Fig) before any supervised panel selection.
- The goal is separating S. Typhi carriage from S. Paratyphi A carriage — that requires the multi-class OPLS-DA model along the second component (p = 0.0031), not the combined-carriage two-class panel.
- The input matrix is not preprocessed GCxGC-TOFMS plasma data or the class labels for carriage vs non-carriage are missing, so supervised OPLS-DA fitting and cross-validated ROC scoring cannot be performed.

## Inputs

- Preprocessed plasma GCxGC-TOFMS metabolite matrix (195 metabolites x 37 samples; 12 S. Typhi carriers, 5 S. Paratyphi A carriers, 20 non-carriage controls)
- Named shortlist of carriage-specific candidate metabolites (glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399)
- Binary class-label vector (combined Salmonella carriage vs non-carriage controls)

## Outputs

- UV-scaled five-metabolite data matrix
- Two-class OPLS-DA model (SIMCA v14) with cross-validated scores and class-separation p-value (target p = 1.4*10−7, Fig 3C)
- Cross-validated ROC curve with AUC = 0.935 (Fig 3D) and 95% CI from 1,000 bootstrap percentile resamples
- Comparison table of panel AUC vs each individual metabolite's AUC (max individual = 0.876, S2 Fig)

## How to apply

Start from the preprocessed plasma metabolite matrix (195 metabolites x 37 samples: 12 S. Typhi carriers, 5 S. Paratyphi A carriers, 20 sterile-bile non-carriage controls) and subset the carriage-specific features — here glutaric acid, hexanoic acid, Unknown_087, Unknown_118, and Unknown_399, chosen for being elevated in chronic carriage but not acute infection. Center and unit-variance (UV) scale the five-column subset, matching the scaling applied to all OPLS-DA models in the study. Fit a two-class OPLS-DA model (combined carriage vs non-carriage) in SIMCA version 14 (Umetrics), using the latent significance concept for covariance loadings and seven-fold cross-validation, then construct an ROC curve in MATLAB (R2014b) from the cross-validated OPLS-DA scores and compute the AUC. The rationale, per the article's hypothesis, is that a combination of co-varying metabolites provides more sensitive and specific biomarkers of carriage than any single metabolite: the panel must therefore be judged against individual-metabolite AUCs (max individual AUC = 0.876, S2 Fig), which the panel exceeded (AUC = 0.935). Quantify panel significance by the model's class separation p-value (reported p = 1.4*10−7, threshold p ≤ 0.05) and report 95% AUC confidence intervals from bootstrap percentile resampling with 1,000 bootstraps.

## Related tools

- **GCxGC-TOFMS** (Source instrument producing the 195-metabolite plasma feature matrix from which the five-metabolite panel is subset.)
- **SIMCA** (Used (version 14, Umetrics) to fit the two-class OPLS-DA panel model with latent significance covariance loadings and seven-fold cross-validation.)
- **MATLAB** (Used (R2014b) to construct the ROC curve from cross-validated OPLS-DA scores, compute the AUC, and bootstrap 1,000 percentile resamples for 95% CI.)

## Evaluation signals

- The two-class OPLS-DA model separates carriage from controls with p ≤ 0.05; the reported target is p = 1.4*10−7 (Fig 3C).
- Cross-validated ROC AUC ≈ 0.935 (Fig 3D), which must exceed the AUC of every individual metabolite (max individual AUC = 0.876, S2 Fig).
- Panel is exactly the five carriage-specific metabolites: glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399.
- Scaling matches the study convention: centered and unit-variance (UV) scaled like all OPLS-DA models.
- Validation uses seven-fold cross-validation in SIMCA and the AUC 95% CI comes from 1,000 bootstrap percentile resamples.

## Limitations

- Small sample size (37 samples: 12 S. Typhi, 5 S. Paratyphi A, 20 controls) limits robustness of the panel.
- No independent validation cohort was available — 'We suggest that resulting metabolite profiles should be validated in an additional, independent cohort. However, this was not possible in this setting'.
- Three of the five panel metabolites are of unknown identity, hindering biological interpretation; the carboxylic acids' relevance is uncertain ('We are uncertain of the relevance of these carboxylic acids').
- Possible confounding from underlying gallbladder conditions, and no controls with other bacterial gallbladder carriage to demonstrate Salmonella specificity.

## Evidence

- [abstract] Panel beats single markers: "Salmonella carriers could be significantly distinguished from non-carriage controls using only five metabolites"
- [intro] Rationale for co-varying panel: "we hypothesize that a combination of co-varying metabolites could potentially provide a means for more sensitive and specific biomarkers of typhoid carriage"
- [supplementary] ROC evaluation procedure: "Panels A-E are showing ROC curves with false positive rates (i.e. 1-specificity) and true positive rates (i.e. sensitivity) on the x- and y-axes respectively"
