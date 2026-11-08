---
name: cross-validated-oplsda-score-scoring
description: Use when you have cross-validated supervised model (e.g. OPLS-DA) scores
  for predefined pairwise class contrasts (carriage vs controls, serovar vs serovar)
  and need ROC curves, AUC values, and bootstrap CIs to report diagnostic performance.
when_to_use_negative:
- Input is unsupervised output (e.g. the PCA scores used only to show sample-group
  distribution) rather than class-supervised OPLS-DA scores — ROC/AUC scoring requires
  labeled pairwise contrasts.
- The goal is univariate biomarker ranking of individual metabolites rather than evaluating
  a multivariate latent-biomarker pattern model.
- No cross-validated scores are available (e.g. only in-sample fitted scores) — AUCs
  would be optimistically biased and not comparable to the paper's reported values.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0219
tools:
- name: Matlab
  role: Used to construct ROC curves (FPR vs TPR) from cross-validated OPLS-DA scores
    and compute AUC values with bootstrap percentile confidence intervals
- name: SIMCA
  role: Source of the cross-validated OPLS-DA model scores that feed the ROC analysis
- name: GCxGC-TOFMS
  role: Platform that generated the 195 plasma metabolite features underlying the
    OPLS-DA models being scored
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:4bb68ae3eab44e9d33686004fec65aded5ab302b236e99b37f84882bbf7782f3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  - Matlab
  - SIMCA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# cross-validated-oplsda-score-scoring

## Summary

Builds ROC curves and computes AUC values with bootstrap 95% confidence intervals from cross-validated OPLS-DA scores to quantify diagnostic performance of metabolite-pattern models, as done for the Salmonella carriage GCxGC-TOFMS models in Fig 2 of the typhoid carriage biomarker study.

## When to use

Use when you have cross-validated supervised model (e.g. OPLS-DA) scores for predefined pairwise class contrasts (carriage vs controls, serovar vs serovar) and need ROC curves, AUC values, and bootstrap CIs to report diagnostic performance.

## When NOT to use

- Input is unsupervised output (e.g. the PCA scores used only to show sample-group distribution) rather than class-supervised OPLS-DA scores — ROC/AUC scoring requires labeled pairwise contrasts.
- The goal is univariate biomarker ranking of individual metabolites rather than evaluating a multivariate latent-biomarker pattern model.
- No cross-validated scores are available (e.g. only in-sample fitted scores) — AUCs would be optimistically biased and not comparable to the paper's reported values.

## Inputs

- Cross-validated OPLS-DA score vectors for each pairwise class contrast (from SIMCA models built on 195 GCxGC-TOFMS metabolite features)
- Class labels defining the pairwise contrasts (carriage vs non-carriage controls; S. Typhi vs S. Paratyphi A carriage)
- Bootstrap specification (1,000 resamples, percentile method)

## Outputs

- ROC curves per pairwise model (false positive rate vs true positive rate), reproducing Fig 2A-2D
- AUC value per model with bootstrap-derived 95% confidence interval
- auc_results.csv — structured table of AUC values and confidence intervals

## How to apply

Load the cross-validated scores from each pairwise OPLS-DA model — here the four contrasts: combined Salmonella carriage vs non-carriage controls, S. Typhi carriage vs controls, S. Paratyphi A carriage vs controls, and S. Typhi vs S. Paratyphi A carriage. In Matlab, build each ROC curve plotting false positive rate (1-specificity) on the x-axis against true positive rate (sensitivity) on the y-axis, replicating Fig 2A-2D. Compute the area under each curve, where 0.5 represents a random classifier and 1 a perfect classifier. Estimate 95% confidence intervals for each AUC by bootstrap percentile resampling with 1,000 bootstraps. Write the AUC values and confidence intervals to a structured table (auc_results.csv) and check them against the paper's acceptance criteria: AUC >= 0.95 for the three carriage-vs-control models and AUC = 0.833 for the weaker S. Typhi vs S. Paratyphi A serovar-comparison model.

## Related tools

- **Matlab** (Used to construct ROC curves (FPR vs TPR) from cross-validated OPLS-DA scores and compute AUC values with bootstrap percentile confidence intervals)
- **SIMCA** (Source of the cross-validated OPLS-DA model scores that feed the ROC analysis)
- **GCxGC-TOFMS** (Platform that generated the 195 plasma metabolite features underlying the OPLS-DA models being scored)

## Evaluation signals

- Each reproduced carriage-vs-control AUC meets the acceptance criterion of >= 0.95 (Fig 2A-2C).
- The S. Typhi vs S. Paratyphi A serovar-comparison AUC reproduces 0.833 (Fig 2D).
- All reproduced AUCs fall within their bootstrap-derived 95% confidence intervals computed with 1,000 bootstraps.
- auc_results.csv contains one row per pairwise model (4 models) with AUC value and CI columns.
- ROC curve axes match the paper's convention: false positive rate (1-specificity) on x, true positive rate (sensitivity) on y.

## Limitations

- The serovar-comparison model (S. Typhi vs S. Paratyphi A) is weaker (AUC 0.833) than the carriage-vs-control models, so acceptance thresholds must be contrast-specific.
- The underlying study had a small sample size and no independent validation cohort, so AUCs may not generalize to other endemic settings.
- The metabolite profiles are descriptive; performance metrics do not establish the physiological meaning of the latent biomarker pattern.

## Evidence

- [supplementary] ROC curves with false positive rates (i.e. 1-specificity) and true positive rates (i.e. sensitivity) on the x- and y-axes respectively: "Panels A-E are showing ROC curves with false positive rates (i.e. 1-specificity) and true positive rates (i.e. sensitivity) on the x- and y-axes respectively"
- [results] AUC >= 0.95 for carriage-vs-control models; 0.833 for serovar comparison: [evidence span withheld — anchor status: not_found]
- [supplementary] Separation of S. Typhi from S. Paratyphi A carriage along the second OPLS-DA component: "the separation of S. Typhi carriage samples from the S. Paratyphi A carriage samples along the second component (p = 0.0031)"
- [intro] Latent biomarker pattern as chemometric strategy: "our primary strategy was to use chemometric bioinformatics to investigate if carriers could be differentiated from non-carriers, thus generating a biomarker pattern (latent biomarker) of carriage"
- [abstract] Discrimination of Salmonella carriage from non-carriage controls: "We were able to significantly discriminate Salmonella carriage samples from non-carriage control samples."
