---
name: bootstrap-confidence-interval-estimation
description: Use when reporting a cross-validated ROC AUC (or other performance metric)
  from a supervised metabolomics or chemometric classifier — e.g. OPLS-DA scores from
  GCxGC-TOFMS data — and you need a defensible interval around it rather than a single
  point value.
when_to_use_negative:
- The input is raw metabolite feature data (195 metabolites × 37 samples) rather than
  already cross-validated classifier scores — fit the OPLS-DA / PCA model first before
  bootstrapping the AUC.
- You need calibration or hypothesis-testing of the model itself (e.g. CV-ANOVA p
  = 0.0031 or p = 1.4×10−7) rather than uncertainty around a performance metric.
- Sample labels are continuous or multi-class without a defined positive class — ROC
  AUC and its bootstrap CI require a two-class comparison.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MATLAB
  role: Constructs the ROC curves from cross-validated OPLS-DA scores and computes
    the AUC before bootstrapping the confidence intervals
- name: SIMCA
  role: Generates the seven-fold cross-validated OPLS-DA scores whose AUC is then
    qualified with bootstrap CIs
- name: GCxGC-TOFMS
  role: Produced the 195-metabolite plasma feature matrix that feeds the models whose
    AUCs are bootstrapped
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:49df18c0a95d179282fef215b7ba858b60bfb66a95c74c99463ad44300ef65c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  - MATLAB
  - Matlab
  - SIMCA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# bootstrap-confidence-interval-estimation

## Summary

Estimates 95% confidence intervals around ROC AUC (and related classifier metrics) via bootstrap percentile resampling with 1,000 bootstraps, as used to qualify the OPLS-DA carriage-diagnostic AUCs in the Nepali cholecystectomy metabolomics study. Provides the uncertainty quantification that turns point AUC estimates from cross-validated pattern-recognition models into interpretable ranges.

## When to use

Use when reporting a cross-validated ROC AUC (or other performance metric) from a supervised metabolomics or chemometric classifier — e.g. OPLS-DA scores from GCxGC-TOFMS data — and you need a defensible interval around it rather than a single point value.

## When NOT to use

- The input is raw metabolite feature data (195 metabolites × 37 samples) rather than already cross-validated classifier scores — fit the OPLS-DA / PCA model first before bootstrapping the AUC.
- You need calibration or hypothesis-testing of the model itself (e.g. CV-ANOVA p = 0.0031 or p = 1.4×10−7) rather than uncertainty around a performance metric.
- Sample labels are continuous or multi-class without a defined positive class — ROC AUC and its bootstrap CI require a two-class comparison.

## Inputs

- Cross-validated classifier scores (e.g. OPLS-DA scores exported from SIMCA v14 for each pairwise carriage-vs-control model)
- Binary class labels (carriage vs non-carriage control, or S. Typhi carriage vs S. Paratyphi A carriage) aligned to those scores
- Number of bootstrap replicates (1,000 in this study) and percentile bounds (2.5 / 97.5)

## Outputs

- Point AUC estimate for each pairwise model (e.g. >= 0.95 for carriage-vs-control ROCs in Fig 2A-2C; 0.833 for the serovar-comparison model in Fig 2D; 0.935 for the five-metabolite model in Fig 3D)
- 95% confidence interval around each AUC, derived from the percentile bootstrap distribution
- ROC curve (true positive rate vs false positive rate) suitable for plotting as in Fig 2A-2D and S2 Fig

## How to apply

After fitting an OPLS-DA model in SIMCA (e.g. combining S. Typhi and S. Paratyphi A carriage vs sterile-bile controls, or the five-metabolite panel of glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399), build the ROC curve in MATLAB (R2014b) from the cross-validated scores with false positive rate (1-specificity) on x and true positive rate on y. Compute the area under that curve, then resample the sample set with replacement 1,000 times and recompute the AUC for each bootstrap replicate; take the 2.5th and 97.5th percentiles of the bootstrap AUC distribution as the 95% confidence interval. This percentile-resampling interval is the accepted way to qualify AUCs reported in the paper — e.g. AUC >= 0.95 for all three carriage-vs-control models and AUC = 0.833 for S. Typhi vs S. Paratyphi A carriage. Decision point: if the 95% CI of a re-derived AUC fails to cover the published value, flag the reproduction as divergent rather than silently accepting it; the CI also guards against over-reading small-cohort differences, which matters here because all models were built on 37 plasma samples (12 S. Typhi carriers, 5 S. Paratyphi A carriers, 20 non-carriage controls).

## Related tools

- **MATLAB** (Constructs the ROC curves from cross-validated OPLS-DA scores and computes the AUC before bootstrapping the confidence intervals)
- **SIMCA** (Generates the seven-fold cross-validated OPLS-DA scores whose AUC is then qualified with bootstrap CIs)
- **GCxGC-TOFMS** (Produced the 195-metabolite plasma feature matrix that feeds the models whose AUCs are bootstrapped)

## Examples

```
auc_ci = bootstrap_percentile_ci(cv_scores, labels, n_boot=1000, lower=2.5, upper=97.5)  # reproduces AUC=0.935, 95% CI for the five-metabolite OPLS-DA model
```

## Evaluation signals

- Reproduced AUC for the combined-carriage model, the S. Typhi model, and the S. Paratyphi A model must each be >= 0.95, matching the published values within their bootstrap 95% CIs
- The S. Typhi vs S. Paratyphi A carriage model must reproduce an AUC of 0.833
- The five-metabolite model must yield AUC ≈ 0.935, exceeding the AUC of every individual metabolite (max individual AUC = 0.876, S2 Fig)
- Each reported AUC must be accompanied by a bootstrap-derived 95% CI computed from 1,000 resamples; a CI that excludes the published AUC indicates a divergent reproduction
- Verification that bootstrap replicates resample samples (not features) with replacement so the AUC distribution reflects between-sample variability in the 37-subject cohort

## Limitations

- The underlying cohort is small (n = 37: 12 S. Typhi carriers, 5 S. Paratyphi A carriers, 20 controls), so the 95% CIs may be wide and are sensitive to resampling of the few carriage samples
- All patients presented with gallbladder conditions, so the derived AUCs and their intervals may not generalise to carriage detected prospectively in other endemic settings
- No independent validation cohort was available; the bootstrap CI quantifies within-cohort uncertainty only, not external validity
- Bootstrap CIs around AUC do not compensate for the lack of controls with other bacterial gallbladder carriage, which limits claims of Salmonella-specificity

## Evidence

- [other] Estimate 95% confidence intervals for each AUC with the bootstrap percentile resampling method using 1,000 bootstraps: [evidence span withheld — anchor status: not_found]
- [other] For all pairwise OPLS-DA models comparing Salmonella carriage samples and non-carriage controls the ROC curves indicated strong diagnostic potential with AUC values >= 0.95; the weaker model comparing S. Typhi and S. Paratyphi A carriage samples had an AUC value of 0.833.: [evidence span withheld — anchor status: not_found]
- [discussion] the major limitation of this study was the small sample size: "The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella"
- [supplementary] ROC curve analysis described in the supplementary: "Panels A-E are showing ROC curves with false positive rates (i.e. 1-specificity) and true positive rates (i.e. sensitivity) on the x- and y-axes respectively"
