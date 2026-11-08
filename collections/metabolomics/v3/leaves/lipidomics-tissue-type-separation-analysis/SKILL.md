---
name: lipidomics-tissue-type-separation-analysis
description: Use when you have paired tissue lipidomics samples (e.g., SAT and VAT
  from the same patients), patient metadata for fixed/random effects, and you need
  to verify that LMM adjustment improves tissue-type separation in PCA.
when_to_use_negative:
- Data are unpaired or lack patient metadata (Sex, Age, BMI, Stage, Tissue) needed
  to specify the LMM fixed effects and the '(1|Id)' random intercept
- The goal is class prediction or feature selection (VIP > 1) via PLS-DA/OPLS-DA rather
  than unsupervised assessment of variance structure via PCA
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: lmm2met
  role: R package providing fitLmm to adjust each metabolite signal with a linear
    mixed-effects model and produce the fitted data matrix
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: Underlying linear mixed-model fitting engine used by lmm2met
- name: ggplot2
  role: Draws PC1-vs-PC2 score plots with stat_ellipse colored by tissue type
- name: R
  role: Analysis environment; PCA is performed with prcomp(center = TRUE, scale. =
    TRUE)
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:46ed00d6fc9a25d054cd573c4905e9c25b0323a80776c6e5441278d4084fa289
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - ggplot2
  - lme4
  - lmm2met
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# lipidomics-tissue-type-separation-analysis

## Summary

Assess whether LMM-based data processing improves the intrinsic variability structure of tissue lipidomics data by running PCA before and after LMM fitting and comparing PC1-explained variance and SAT/VAT separation.

## When to use

Use when you have paired tissue lipidomics samples (e.g., SAT and VAT from the same patients), patient metadata for fixed/random effects, and you need to verify that LMM adjustment improves tissue-type separation in PCA.

## When NOT to use

- Data are unpaired or lack patient metadata (Sex, Age, BMI, Stage, Tissue) needed to specify the LMM fixed effects and the '(1|Id)' random intercept
- The goal is class prediction or feature selection (VIP > 1) via PLS-DA/OPLS-DA rather than unsupervised assessment of variance structure via PCA

## Inputs

- Adipose lipidomics data matrix (158 lipids x paired SAT/VAT samples from 59 CRC patients), log2-transformed
- Patient metadata table with Sex, Age, BMI, Stage, Location, Tissue, and patient Id

## Outputs

- PC1-vs-PC2 PCA score plots (original vs. LMM-fitted) colored by tissue type with confidence ellipses
- PC1 explained variance proportions for both PCA runs and their difference (~6 percentage points)
- LMM-fitted data matrix (fitMet$fittedDat)

## How to apply

Load the packaged adipose lipid dataset (158 lipids, 59 CRC patients, paired subcutaneous and visceral tissue) from the lmm2met R package. Fit a linear mixed-effects model to each metabolite with fitLmm using fixed effects Sex, Age, BMI, Stage, Location and Tissue, and a random intercept '(1|Id)' for patient identifier. Run PCA with prcomp(center = TRUE, scale. = TRUE) on the original log2-transformed data and on the LMM-processed data (fitMet$fittedDat), drawing PC1-vs-PC2 score plots colored by tissue type with ggplot2 and stat_ellipse. The rationale is that LMM fitting systematically minimizes subject-specific variation, so the LMM-fitted PCA should explain more variance on PC1 and separate SAT from VAT more cleanly than the original data.

## Related tools

- **lmm2met** (R package providing fitLmm to adjust each metabolite signal with a linear mixed-effects model and produce the fitted data matrix) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (Underlying linear mixed-model fitting engine used by lmm2met)
- **ggplot2** (Draws PC1-vs-PC2 score plots with stat_ellipse colored by tissue type)
- **R** (Analysis environment; PCA is performed with prcomp(center = TRUE, scale. = TRUE)) — https://www.r-project.org/

## Examples

```
install_github('kwanjeeraw/lmm2met'); fitMet <- fitLmm(dat, fixed=c('Sex','Age','BMI','Stage','Location','Tissue'), random='(1|Id)'); pca <- prcomp(t(fitMet$fittedDat), center=TRUE, scale.=TRUE)
```

## Evaluation signals

- PC1 explained variance of the LMM-fitted PCA is approximately 6 percentage points higher than that of the original (unfitted) data
- PC1-vs-PC2 score plot of LMM-fitted data shows visibly better SAT/VAT separation (tighter ellipses per tissue type) than the original plot
- Subject-wise boxplots after LMM fitting have medians closer to zero and less within-tissue-type variability
- Both PCA runs use identical settings: prcomp with center = TRUE and scale. = TRUE on log2-transformed matrices
- Model formula for every metabolite includes fixed effects Sex, Age, BMI, Stage, Location, Tissue and random intercept (1|Id)

## Limitations

- Increased sensitivity of LMM processing may introduce false positives into downstream analyses
- Performance claims are preliminary; the authors recommend further experiments to validate results
- Dataset is paired tissue samples with per-patient random intercepts; conclusions may not transfer to unpaired designs

## Evidence

- [results] PCA of LMM-fitted data showed better separation between SAT and VAT with 6% increase in explanation of variation on PC1 compared to original data: "depicted better separation between SAT and VAT with 6% increase in explanation of variation on PC1"
- [methods] LMM data processing step precedes PCA: "we propose an additional data processing step using linear mixed-effects modelling to readjust an individual metabolite signal prior to multivariate analyses."
- [methods] Log2 transformation of input data: "Both datasets were pre-processed as described previously [21,27] and log2-transformed prior to use in this study."
- [readme] lmm2met package availability and purpose: "Linear mixed-effects modelling for normalization of clinical metabolomics data by using subject metada."
- [results] Variance reduction rationale after LMM fitting: "variance of a metabolite across all patients was substantially reduced and outliers were corrected after LMM fitting"
