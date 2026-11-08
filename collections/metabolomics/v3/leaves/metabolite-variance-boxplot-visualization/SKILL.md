---
name: metabolite-variance-boxplot-visualization
description: Use when you have a metabolomics matrix and an lmm2met fit object and
  need to visually validate that linear mixed-effects modelling reduced per-metabolite
  variance, as in Figure S-1/Fig. 3a.
when_to_use_negative:
- Input data are raw, non-log2-transformed signals that have not gone through the
  LMM fitting step — the before/after comparison is only meaningful on log2-transformed
  data and its fitted counterpart
- The goal is statistical significance testing of metabolite differences rather than
  visual QC of variance reduction; use PLS-DA/OPLS-DA with VIP > 1 feature selection
  instead
- There is no subject metadata available to formulate the LMMs — the paper's approach
  requires patient metadata as rich information of subject characteristics to model
  metabolite abundances
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: lmm2met
  role: Provides the fit object and the plot(type='fitted') method that renders the
    before-vs-after boxplots using ggplot2
  repo: https://github.com/kwanjeeraw/lmm2met
- name: ggplot2
  role: Graphics backend used by lmm2met's plotting to draw the subject-wise boxplots
    colored by tissue type
- name: lme4
  role: Underlying lmer fitting procedure on which fitLmm's per-metabolite LMMs are
    built; the fitted values plotted are its fixed effects plus subject-specific random
    intercepts
- name: R
  role: Runtime environment for all analyses; lmm2met is installed via devtools::install_github
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:ede3e41674620770fcfa6f6a4cf93ff1b1821090ba3dfea9ba084dbbba6d36c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - ggplot2
  - lme4
  - lmm2met
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# metabolite-variance-boxplot-visualization

## Summary

Generates before-vs-after boxplots of metabolite signals (e.g., log2-transformed lipids from SAT and VAT of 59 CRC patients) using ggplot2 via lmm2met, to demonstrate that LMM fitting reduces per-metabolite variance and corrects outliers.

## When to use

Use when you have a metabolomics matrix and an lmm2met fit object and need to visually validate that linear mixed-effects modelling reduced per-metabolite variance, as in Figure S-1/Fig. 3a.

## When NOT to use

- Input data are raw, non-log2-transformed signals that have not gone through the LMM fitting step — the before/after comparison is only meaningful on log2-transformed data and its fitted counterpart
- The goal is statistical significance testing of metabolite differences rather than visual QC of variance reduction; use PLS-DA/OPLS-DA with VIP > 1 feature selection instead
- There is no subject metadata available to formulate the LMMs — the paper's approach requires patient metadata as rich information of subject characteristics to model metabolite abundances

## Inputs

- lmm2met fit object returned by fitLmm (with fittedDat slot), built from a log2-transformed lipid matrix (e.g., 158 lipids from SAT/VAT of 59 CRC patients, Metabolomics Workbench ST000081 or the packaged 'adipose' data)
- Sample metadata used as fixed effects (Sex, Age, BMI, Stage, Location, Tissue) and patient ID as random intercept
- R environment with lmm2met, lme4, and ggplot2 installed

## Outputs

- Before-vs-after boxplots of metabolite signals colored by tissue type (SC vs VS), analogous to Figure S-1 / Fig. 3a
- Visual confirmation of reduced per-metabolite variance and corrected outliers after LMM fitting
- Optional heatmaps of fixed-effect coefficients (type='fixeff') and their significance levels (type='coeff')

## How to apply

After fitting per-metabolite LMMs with lmm2met::fitLmm on the log2-transformed adipose lipid data (158 lipids, fix=c('Sex','Age','BMI','Stage','Location','Tissue'), random='(1|Id)'), extract the processed matrix from fitMet$fittedDat, which incorporates the fixed effects and subject-specific random intercepts built on lme4's lmer procedure. Then render before-vs-after subject-wise boxplots colored by tissue type (SC vs VS) with plot(fitMet, type='fitted', fix='Tissue'), which uses ggplot2. This reproduces the paper's demonstration that per-metabolite variance was substantially reduced and outliers corrected after LMM fitting. Optionally complement with plot(fitMet, type='fixeff') and plot(fitMet, type='coeff') heatmaps of fixed-effect coefficients and their significance to contextualize which metadata terms drove the adjustment.

## Related tools

- **lmm2met** (Provides the fit object and the plot(type='fitted') method that renders the before-vs-after boxplots using ggplot2) — https://github.com/kwanjeeraw/lmm2met
- **ggplot2** (Graphics backend used by lmm2met's plotting to draw the subject-wise boxplots colored by tissue type)
- **lme4** (Underlying lmer fitting procedure on which fitLmm's per-metabolite LMMs are built; the fitted values plotted are its fixed effects plus subject-specific random intercepts)
- **R** (Runtime environment for all analyses; lmm2met is installed via devtools::install_github) — https://www.r-project.org/

## Examples

```
fitMet <- lmm2met::fitLmm(data=adipose, fix=c('Sex','Age','BMI','Stage','Location','Tissue'), random='(1|Id)'); plot(fitMet, type='fitted', fix='Tissue')
```

## Evaluation signals

- Per-metabolite variance is substantially lower in the fitted (after) matrix fitMet$fittedDat than in the original log2-transformed input
- After LMM fitting, subject-wise boxplots have a median closer to zero and show less variability within tissue type (SC vs VS)
- Extreme/outlier metabolite signals present in the before-boxplots are visibly corrected in the after-boxplots
- Str() inspection of the input confirms the expected data shape (158 log2-transformed lipids from SAT and VAT of 59 CRC patients)
- summary(fitMet$completeMod[[1]]) returns a converged lme4 lmer model for the first metabolite, confirming fits are valid before plotting

## Limitations

- The method systematically minimizes subject-specific variation in each metabolite, but with the potential introduction of false positives due to increased sensitivity
- Further experiments are recommended to validate the results; performance claims are qualified as being at a preliminary stage
- The lmm2met repository README provides no changelog; version pinned at 1.0 (12 September 2018)

## Evidence

- [results] Variance reduction finding: "variance of a metabolite across all patients was substantially reduced and outliers were corrected after LMM fitting"
- [results] Subject-wise boxplot behavior: "the subject-wise boxplots after LMM fitting had a median closer to zero and showed less variability within tissue type"
- [methods] LMM processing step definition: "We developed the R package lmm2met to process a metabolomics matrix with the LMM approach."
- [readme] Package purpose from README: "Linear mixed-effects modelling for normalization of clinical metabolomics data by using subject metada."
- [abstract] Rationale for using patient metadata: "we highlight the importance of patient metadata as it contains rich information of subject characteristics, which can be used to model and normalize metabolite abundances."
