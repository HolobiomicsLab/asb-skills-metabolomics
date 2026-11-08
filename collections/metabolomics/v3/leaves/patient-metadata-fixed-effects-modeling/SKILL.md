---
name: patient-metadata-fixed-effects-modeling
description: Use when a metabolomics feature matrix has paired or repeated measurements
  per patient plus subject metadata, and residual inter-individual variation masks
  biological group separation (e.g. SAT vs VAT) prior to PCA or PLS-DA.
when_to_use_negative:
- The data lack patient metadata or repeated/paired measurements per subject, so no
  random intercept '(1|Id)' can be defined
- A downstream classifier evaluation pipeline requires untouched raw intensities rather
  than model-readjusted signals
- The matrix is already processed by an equivalent normalization that accounts for
  subject-specific covariates
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: lmm2met
  role: R package that implements the LMM data-processing step, exposing fitLmm and
    plotting methods
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: Underlying lmer model-fitting procedure on which lmm2met is implemented
- name: R
  role: Runtime environment for all analyses
- name: ggplot2
  role: Visualization of before-vs-after boxplots and PCA score plots
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:9d1fe00857f8e791d8bad8d497566d772e021c891ba2cecad6a91be6b0f6ae6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - ggplot2
  - lme4
  - lmm2met
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# patient-metadata-fixed-effects-modeling

## Summary

Fit a linear mixed-effects model (LMM) to each metabolite in a clinical metabolomics matrix, using patient metadata (Sex, Age, BMI, Stage, Location, Tissue) as fixed effects and a per-patient random intercept, via the lmm2met R package. This readjusts individual metabolite signals to reduce subject-specific variation before multivariate analyses such as PCA or PLS-DA.

## When to use

Use when a metabolomics feature matrix has paired or repeated measurements per patient plus subject metadata, and residual inter-individual variation masks biological group separation (e.g. SAT vs VAT) prior to PCA or PLS-DA.

## When NOT to use

- The data lack patient metadata or repeated/paired measurements per subject, so no random intercept '(1|Id)' can be defined
- A downstream classifier evaluation pipeline requires untouched raw intensities rather than model-readjusted signals
- The matrix is already processed by an equivalent normalization that accounts for subject-specific covariates

## Inputs

- Log2-transformed metabolomics feature matrix (e.g. 158 lipids x 118 samples: SAT and VAT pairs from 59 CRC patients)
- Patient metadata table (Sex, Age, BMI, Stage, Location, patient Id) used to formulate the LMMs
- Sample-to-patient mapping enabling paired/repeated-measures structure

## Outputs

- LMM-fitted (readjusted) metabolomics matrix via fitMet$fittedDat
- Per-metabolite LMM objects in fitMet$completeMod for inspection with summary()
- Heatmaps of fixed-effect coefficients and their significance levels
- Before-vs-after per-metabolite boxplots colored by tissue type showing variance reduction

## How to apply

Load the log2-transformed metabolomics matrix (e.g. the lmm2met 'adipose' dataset of 158 lipids from SAT and VAT of 59 CRC patients, Metabolomics Workbench ST000081) and inspect it with str(). Fit an LMM per metabolite with lmm2met::fitLmm using fixed effects fix=c('Sex','Age','BMI','Stage','Location','Tissue') and random='(1|Id)' for the patient identifier, built on lme4's lmer procedure. Extract the processed matrix from fitMet$fittedDat, which accounts for the fixed effects and subject-specific random intercepts; inspect the first metabolite's model with summary(fitMet$completeMod[[1]]). Visualize fixed-effect coefficients and their significance with plot(fitMet, type='fixeff') and plot(fitMet, type='coeff'), and compare before-vs-after boxplots with plot(fitMet, type='fitted', fix='Tissue'). The rationale is that patient metadata contains rich subject-characteristic information that can be used to model and systematically minimize subject-specific variation in each metabolite signal.

## Related tools

- **lmm2met** (R package that implements the LMM data-processing step, exposing fitLmm and plotting methods) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (Underlying lmer model-fitting procedure on which lmm2met is implemented)
- **R** (Runtime environment for all analyses)
- **ggplot2** (Visualization of before-vs-after boxplots and PCA score plots)

## Examples

```
install_github('kwanjeeraw/lmm2met'); fitMet <- fitLmm(adipose, fix=c('Sex','Age','BMI','Stage','Location','Tissue'), random='(1|Id)')
```

## Evaluation signals

- Per-metabolite variance across all patients is substantially reduced after LMM fitting, with subject-wise boxplots showing a median closer to zero and less variability within tissue type
- PCA on the LMM-fitted matrix (prcomp, center=TRUE, scale.=TRUE) yields a PC1 explained-variance proportion approximately 6 percentage points higher than PCA on the original data
- PC1-vs-PC2 score plot of the LMM-fitted data shows better separation between SAT and VAT, with stat_ellipse group boundaries less overlapping
- summary(fitMet$completeMod[[1]]) returns a converged lmer model including the specified fixed effects and the random intercept term
- Dimensions of fitMet$fittedDat match the original matrix (158 metabolites, paired tissue samples of 59 patients)

## Limitations

- Increased sensitivity may come with the potential introduction of false positives
- Further experiments are recommended to validate the results (validation gap)
- Performance claims are qualified as being at a preliminary current stage

## Evidence

- [abstract] abstract: "we propose an additional data processing step using linear mixed-effects modelling to readjust an individual metabolite signal prior to multivariate analyses."
- [methods] methods: "The package was implemented on a linear model-fitting procedure of the lme4 package"
- [results] results: "variance of a metabolite across all patients was substantially reduced and outliers were corrected after LMM fitting"
- [readme] readme: "Linear mixed-effects modelling for normalization of clinical metabolomics data by using subject metada."
