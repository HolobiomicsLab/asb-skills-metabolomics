---
name: seven-fold-cross-validation-model-assessment
description: Use when a small, high-dimensional metabolomics matrix (e.g. 195 metabolites
  x 37 plasma samples) must be assessed for class separation (carriage vs control)
  with an honest generalization estimate rather than a training-set fit.
when_to_use_negative:
- The question is unsupervised exploratory structure (e.g. sample-group distribution)
  rather than supervised class prediction — use PCA scores instead, as in panel A
  of the study.
- Comparing a single metabolite between groups — individual metabolites were assessed
  separately (max individual AUC = 0.876) and do not require cross-validated multivariate
  modeling.
- The cohort is large enough for a true independent validation set; the article notes
  no independent validation cohort was possible here due to the small sample size,
  so cross-validation was the only available honesty check.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3173
tools:
- name: SIMCA
  role: Fit the two-class OPLS-DA model with UV scaling, latent significance covariance
    loadings, and seven-fold cross-validation (version 14, Umetrics)
- name: MATLAB
  role: Construct the ROC curve from cross-validated OPLS-DA scores, compute the AUC,
    and run 1,000-bootstrap percentile resampling for the 95% CI (R2014b)
- name: GCxGC-TOFMS
  role: Two-dimensional gas chromatography coupled with time-of-flight MS that generated
    the 195-metabolite plasma feature matrix fed into the cross-validated model
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:a3e0e834cf4141445cac4121f7a46612a7aa3afe223694788da923ab55dd9433
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  - MATLAB
  - SIMCA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# seven-fold-cross-validation-model-assessment

## Summary

A supervised model-validation skill in which a two-class OPLS-DA classifier is fit on UV-scaled GCxGC-TOFMS metabolite features and assessed by seven-fold cross-validation, with diagnostic performance quantified as a cross-validated ROC AUC and a bootstrap-percentile 95% CI. It was used to show that a five-metabolite panel (glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399) separates Salmonella carriage from non-carriage controls.

## When to use

Use when a small, high-dimensional metabolomics matrix (e.g. 195 metabolites x 37 plasma samples) must be assessed for class separation (carriage vs control) with an honest generalization estimate rather than a training-set fit.

## When NOT to use

- The question is unsupervised exploratory structure (e.g. sample-group distribution) rather than supervised class prediction — use PCA scores instead, as in panel A of the study.
- Comparing a single metabolite between groups — individual metabolites were assessed separately (max individual AUC = 0.876) and do not require cross-validated multivariate modeling.
- The cohort is large enough for a true independent validation set; the article notes no independent validation cohort was possible here due to the small sample size, so cross-validation was the only available honesty check.

## Inputs

- Preprocessed GCxGC-TOFMS plasma metabolite matrix (195 metabolites x 37 samples: 12 S. Typhi carriers, 5 S. Paratyphi A carriers, 20 non-carriage controls)
- Subset of candidate biomarker features (five metabolites: glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399)
- Binary class labels (combined Salmonella carriage vs non-carriage controls)

## Outputs

- Seven-fold cross-validated two-class OPLS-DA model (SIMCA v14) with class-separation significance p-value (reported p = 1.4*10^-7)
- Cross-validated ROC curve with AUC (reported AUC = 0.935)
- 95% confidence interval for the AUC via 1,000-bootstrap percentile resampling

## How to apply

Center and unit-variance (UV) scale the metabolite matrix (here the five carriage-specific features glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399 from a 195-metabolite x 37-sample GCxGC-TOFMS dataset), then fit a two-class OPLS-DA model (combined S. Typhi + S. Paratyphi A carriage vs sterile-bile non-carriage controls) in SIMCA v14 using the latent significance concept for covariance loadings and seven-fold cross-validation. Export the cross-validated scores and construct a ROC curve in MATLAB (R2014b), computing the area under the curve. Estimate 95% confidence intervals for the AUC by bootstrap percentile resampling with 1,000 bootstraps. Judge success by a CV-ANOVA-style significance threshold of p <= 0.05 (reported p = 1.4*10^-7) and an AUC near the reported 0.935, which must exceed the best single-metabolite AUC (0.876, S2 Fig).

## Related tools

- **SIMCA** (Fit the two-class OPLS-DA model with UV scaling, latent significance covariance loadings, and seven-fold cross-validation (version 14, Umetrics))
- **MATLAB** (Construct the ROC curve from cross-validated OPLS-DA scores, compute the AUC, and run 1,000-bootstrap percentile resampling for the 95% CI (R2014b))
- **GCxGC-TOFMS** (Two-dimensional gas chromatography coupled with time-of-flight MS that generated the 195-metabolite plasma feature matrix fed into the cross-validated model)

## Examples

```
[X,Y,T,AUC] = perfcurve(carriageLabels, cvOplsdaScores, 1); aucCI = bootci(1000, {@(s,l)perfcurve(l,s,1), cvOplsdaScores, carriageLabels});
```

## Evaluation signals

- Model separates carriage from non-carriage controls with p <= 0.05 (study reported p = 1.4*10^-7, Fig 3C).
- Cross-validated ROC AUC approximates 0.935 (Fig 3D).
- Panel AUC (0.935) exceeds the AUC of every individual metabolite (max individual AUC = 0.876, S2 Fig).
- All five metabolites (glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399) are present in the model after UV scaling of the 195 x 37 matrix.
- Bootstrap CI for the AUC is computed from 1,000 percentile-resampled replicates, not from the point estimate alone.

## Limitations

- The major limitation of this study was the small sample size (37 samples), which constrains the stability of cross-validated performance estimates.
- No independent validation cohort was available; resulting metabolite profiles should be validated in an additional, independent cohort.
- All patients presented with gallbladder conditions, so the primary cause for cholecystectomy may confound the metabolite signatures driving the model.
- Lack of controls with other bacterial gallbladder carriage means the five-metabolite model cannot yet be claimed as truly Salmonella-specific.
- The physiological meaning of the detected metabolite profiles is unclear; further epidemiological investigations in alternative endemic enteric fever settings are required.

## Evidence

- [abstract] We were able to significantly discriminate Salmonella carriage samples from non-carriage control samples.: "We were able to significantly discriminate Salmonella carriage samples from non-carriage control samples."
- [supplementary] Panels A-E are showing ROC curves with false positive rates (i.e. 1-specificity) and true positive rates (i.e. sensitivity) on the x- and y-axes respectively: "Panels A-E are showing ROC curves with false positive rates (i.e. 1-specificity) and true positive rates (i.e. sensitivity) on the x- and y-axes respectively"
- [supplementary] models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
- [discussion] The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella: "The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella"
