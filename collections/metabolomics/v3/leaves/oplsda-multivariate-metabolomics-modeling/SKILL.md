---
name: oplsda-multivariate-metabolomics-modeling
description: Use when the goal is to distinguish two or more pre-defined phenotypic
  classes (e.g., carriage vs. non-carriage, S. Typhi vs. S.
when_to_use_negative:
- When the goal is exploratory, unsupervised structure discovery without pre-defined
  class labels — the paper uses PCA for that (Fig 1A-style panel), not OPLS-DA.
- When samples lack a pre-defined binary or multiclass phenotype (e.g., only carriage
  samples with no non-carriage controls), since supervised OPLS-DA requires labeled
  classes.
- When an independent validation cohort is available and the aim is external validation
  rather than model fitting — the paper notes its model was not validated on an independent
  cohort, so treat in-sample AUC as provisional.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GCxGC-TOFMS
  role: Two-dimensional gas chromatography coupled with time-of-flight mass spectrometry
    used to generate the 195-metabolite plasma feature matrix that is the direct input
    to OPLS-DA.
- name: SIMCA
  role: Umetrics SIMCA (version 14) used to fit the supervised OPLS-DA models with
    UV scaling and seven-fold cross-validation.
- name: MATLAB
  role: MATLAB (R2014b) used to construct ROC curves from cross-validated OPLS-DA
    scores and compute AUC with 1,000-bootstrap 95% confidence intervals.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:9b53dea2d5089dd5271a0bd3d6515bfaebad7935fb5d5fac2a1916abb981103e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  - MATLAB
  - SIMCA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# oplsda-multivariate-metabolomics-modeling

## Summary

Fit supervised orthogonal partial least-squares discriminant analysis (OPLS-DA) models on GCxGC-TOFMS plasma metabolite matrices to separate chronic Salmonella carriage from non-carriage controls and, optionally, S. Typhi from S. Paratyphi A carriage, then assess diagnostic power with cross-validated ROC curves and bootstrap-estimated AUCs.

## When to use

Use when the goal is to distinguish two or more pre-defined phenotypic classes (e.g., carriage vs. non-carriage, S. Typhi vs. S. Paratyphi A) from a GCxGC-TOFMS metabolite abundance matrix, especially when a small subset of co-varying metabolites is suspected to outperform any single-metabolite marker.

## When NOT to use

- When the goal is exploratory, unsupervised structure discovery without pre-defined class labels — the paper uses PCA for that (Fig 1A-style panel), not OPLS-DA.
- When samples lack a pre-defined binary or multiclass phenotype (e.g., only carriage samples with no non-carriage controls), since supervised OPLS-DA requires labeled classes.
- When an independent validation cohort is available and the aim is external validation rather than model fitting — the paper notes its model was not validated on an independent cohort, so treat in-sample AUC as provisional.

## Inputs

- Preprocessed GCxGC-TOFMS plasma metabolite abundance matrix (metabolites × samples; 195 metabolites × 37 samples in the reference study)
- Sample class labels (S. Typhi carriage, S. Paratyphi A carriage, non-carriage control)
- Optional: list of selected candidate metabolites for a reduced diagnostic panel (e.g., glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399)

## Outputs

- OPLS-DA scores/loadings plot showing separation of non-carriage controls from Salmonella carriage along the first component and S. Typhi from S. Paratyphi A along the second component (Fig 1B-style panel)
- Cross-validated class-separation p-value (e.g., p = 0.0031 for three-group model; p = 1.4×10⁻⁷ for the five-metabolite carriage vs. control model)
- ROC curve (false-positive rate vs. true-positive rate) built from cross-validated OPLS-DA scores with AUC and 95% bootstrap CI (AUC ≈ 0.935 for the five-metabolite model)

## How to apply

Load the preprocessed GCxGC-TOFMS plasma metabolite matrix (in this study 195 metabolites × 37 samples: 12 S. Typhi carriers, 5 S. Paratyphi A carriers, and 20 sterile-bile non-carriage controls) and center plus unit-variance (UV) scale the data before fitting. Fit a two-class OPLS-DA model (e.g., combined carriage vs. non-carriage controls) in SIMCA (version 14, Umetrics) using seven-fold cross-validation and the latent significance concept for model covariance loadings. Decision point: for a minimal diagnostic panel, subset to the metabolites elevated in chronic carriage but not acute infection — here glutaric acid, hexanoic acid, Unknown_087, Unknown_118, and Unknown_399 — and confirm the five-metabolite model still separates classes significantly (p ≤ 0.05). Then export cross-validated OPLS-DA scores to MATLAB (R2014b) to build a ROC curve and compute the area under the curve, estimating 95% confidence intervals by bootstrap percentile resampling with 1,000 bootstraps. Rationale: the paper hypothesizes that a combination of co-varying metabolites provides more sensitive and specific carriage biomarkers than single metabolites, and reports a five-metabolite model achieving p = 1.4×10⁻⁷ and AUC = 0.935 versus a maximum individual-metabolite AUC of 0.876.

## Related tools

- **GCxGC-TOFMS** (Two-dimensional gas chromatography coupled with time-of-flight mass spectrometry used to generate the 195-metabolite plasma feature matrix that is the direct input to OPLS-DA.)
- **SIMCA** (Umetrics SIMCA (version 14) used to fit the supervised OPLS-DA models with UV scaling and seven-fold cross-validation.)
- **MATLAB** (MATLAB (R2014b) used to construct ROC curves from cross-validated OPLS-DA scores and compute AUC with 1,000-bootstrap 95% confidence intervals.)

## Examples

```
SIMCA 14: UV-scale 195-metabolite matrix → fit OPLS-DA (carriage vs. control, 7-fold CV) → export CV scores → MATLAB R2014b: ROC + 1000-bootstrap AUC CI.
```

## Evaluation signals

- Permutation/cross-validated separation p-value must reach significance at p ≤ 0.05 (reference study reports p = 0.0031 for the three-group model and p = 1.4×10⁻⁷ for the five-metabolite carriage vs. control model).
- ROC AUC from cross-validated scores should exceed the best individual metabolite's AUC (in the study, the five-metabolite model reached 0.935 vs. a maximum single-metabolite AUC of 0.876).
- Model must be fitted on UV-scaled data and use seven-fold cross-validation in SIMCA, matching the paper's preprocessing and validation protocol.
- AUC 95% confidence interval must be derived from 1,000 bootstrap percentile resamples, not a normal approximation.
- Scores plot should show non-carriage controls separated from Salmonella carriage along the first predictive component, mirroring the paper's reported OPLS-DA structure.

## Limitations

- Small sample size (37 cholecystectomy patients) limits robustness of the fitted model.
- No independent validation cohort — all performance metrics are in-sample/cross-validated only.
- All patients presented with gallbladder conditions, so underlying cholecystectomy indications may confound the metabolite signature.
- No controls with other bacterial gallbladder carriage, so metabolites cannot be confirmed as truly Salmonella-specific.
- The detected metabolite profiles' physiological meaning and the relevance of the carboxylic acid markers are uncertain; the paper recommends validation in other endemic enteric fever populations before any diagnostic test is developed.

## Evidence

- [supplementary] OPLS-DA separated non-carriage controls from Salmonella carriage samples along the first component and S. Typhi carriage from S. Paratyphi A carriage along the second component (p = 0.0031): "the separation of S. Typhi carriage samples from the S. Paratyphi A carriage samples along the second component (p = 0.0031)"
- [supplementary] OPLS-DA model built from five metabolites significantly separated carriage from controls (p = 1.4×10⁻⁷) with cross-validated ROC AUC of 0.935: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
- [abstract] Salmonella carriers could be significantly distinguished from non-carriage controls using only five metabolites: "Salmonella carriers could be significantly distinguished from non-carriage controls using only five metabolites"
- [abstract] Supervised pattern recognition modeling applied to GCxGC-TOFMS metabolite data: "using two-dimensional gas chromatography coupled with time-of-flight mass spectrometry (GCxGC-TOFMS) and supervised pattern recognition modeling"
- [intro] Hypothesis that co-varying metabolites provide more sensitive and specific biomarkers than single metabolites: "we hypothesize that a combination of co-varying metabolites could potentially provide a means for more sensitive and specific biomarkers of typhoid carriage"
