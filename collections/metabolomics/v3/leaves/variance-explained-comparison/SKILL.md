---
name: variance-explained-comparison
description: Use when you have a metabolomics matrix (e.g. the 158-lipid, 59-CRC-patient
  adipose dataset) processed both before and after linear mixed-effects model fitting
  and need to demonstrate improved group separation via PC1 explained variance.
when_to_use_negative:
- The data matrix has not been fit per metabolite with the LMM (fixed effects Sex,
  Age, BMI, Stage, Location, Tissue; random intercept (1|Id)), so no before/after
  contrast exists
- The analysis goal is class prediction or feature selection (VIP > 1) rather than
  demonstrating improved variance structure via PCA
- The dataset lacks paired/subject metadata needed to formulate the LMMs, so the LMM-processed
  counterpart cannot be generated
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: lmm2met
  role: Provides fitLmm to fit each metabolite to an LMM and produce the fitted data
    matrix (fitMet$fittedDat) compared against the original data
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: Underlying linear model-fitting procedure on which the lmm2met LMM fitting
    is implemented
- name: ggplot2
  role: Draws the PC1-vs-PC2 score plots colored by tissue type with stat_ellipse
    for both PCA runs
- name: R
  role: Environment in which PCA (prcomp), LMM fitting, and plotting are performed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:d1d23ece14e77f5ef5b667f9d32f2bffd19738b22a4276be2c19e76e8212131a
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# variance-explained-comparison

## Summary

Quantifies the benefit of LMM-based metabolite-signal readjustment by running PCA on original vs. LMM-fitted metabolomics data and comparing the PC1 explained-variance proportion, validated in the CRC adipose lipid tutorial as a ~6 percentage-point increase with improved SAT/VAT separation.

## When to use

Use when you have a metabolomics matrix (e.g. the 158-lipid, 59-CRC-patient adipose dataset) processed both before and after linear mixed-effects model fitting and need to demonstrate improved group separation via PC1 explained variance.

## When NOT to use

- The data matrix has not been fit per metabolite with the LMM (fixed effects Sex, Age, BMI, Stage, Location, Tissue; random intercept (1|Id)), so no before/after contrast exists
- The analysis goal is class prediction or feature selection (VIP > 1) rather than demonstrating improved variance structure via PCA
- The dataset lacks paired/subject metadata needed to formulate the LMMs, so the LMM-processed counterpart cannot be generated

## Inputs

- Original log2-transformed metabolomics data matrix (e.g. 158 lipids x 59 CRC patients with SAT and VAT tissue pairs)
- LMM-fitted data matrix (fitMet$fittedDat) produced by fitLmm with fixed effects Sex, Age, BMI, Stage, Location, Tissue and random intercept (1|Id)
- Sample metadata (tissue type labels for coloring and ellipse grouping in score plots)

## Outputs

- PC1-vs-PC2 PCA score plots for original and LMM-fitted data colored by tissue type with stat_ellipse
- PC1 and PC2 explained-variance proportions for both PCA runs
- Comparison of PC1 explained variance (expected ~6 percentage-point increase after LMM fitting)

## How to apply

Load the log2-transformed data (e.g. the packaged adipose lipid dataset with 158 lipids from 59 colorectal cancer patients with SAT/VAT pairs) and fit a linear mixed-effects model per metabolite with fitLmm using fixed effects Sex, Age, BMI, Stage, Location and Tissue plus a random intercept '(1|Id)' for patient identifier, obtaining the fitted matrix (fitMet$fittedDat). Run PCA identically on both the original and LMM-fitted matrices using prcomp with center = TRUE and scale. = TRUE. Draw PC1-vs-PC2 score plots with ggplot2, coloring points by tissue type (SAT vs. VAT) and adding stat_ellipse for group separation. Compare the PC1 explained-variance proportions between the two runs: the LMM-fitted PCA should yield a higher PC1 proportion (approximately 6 percentage points higher in the article) and visibly better SAT/VAT separation. Rationale: LMM fitting systematically minimizes subject-specific variation in each metabolite, so a larger PC1 share and tighter ellipses indicate the intrinsic variability structure has improved.

## Related tools

- **lmm2met** (Provides fitLmm to fit each metabolite to an LMM and produce the fitted data matrix (fitMet$fittedDat) compared against the original data) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (Underlying linear model-fitting procedure on which the lmm2met LMM fitting is implemented)
- **ggplot2** (Draws the PC1-vs-PC2 score plots colored by tissue type with stat_ellipse for both PCA runs)
- **R** (Environment in which PCA (prcomp), LMM fitting, and plotting are performed)

## Examples

```
fitMet <- lmm2met::fitLmm(data, fixed='Sex+Age+BMI+Stage+Location+Tissue', random='(1|Id)'); pca_lmm <- prcomp(fitMet$fittedDat, center=TRUE, scale.=TRUE)
```

## Evaluation signals

- PC1 explained-variance proportion of the LMM-fitted PCA is approximately 6 percentage points higher than the PC1 proportion of the original (unfitted) data PCA
- The LMM-fitted score plot shows visibly better SAT/VAT separation (tighter stat_ellipse groups) than the original-data score plot
- Both PCA runs use identical settings (prcomp with center = TRUE, scale. = TRUE) so the variance comparison is apples-to-apples
- Variance of each metabolite across patients is substantially reduced after LMM fitting, consistent with the reported subject-wise boxplots having medians closer to zero and less within-tissue variability

## Limitations

- The ~6% PC1 increase is demonstrated on the adipose lipid dataset (158 lipids, 59 CRC patients) and may differ on other metabolomics datasets
- Increased sensitivity from LMM processing carries the potential introduction of false positives
- Performance claims are qualified as being at the current (preliminary) stage; further experiments are recommended to validate the results

## Evidence

- [results] PCA of LMM-fitted data showed better separation between SAT and VAT with 6% increase in explanation of variation on PC1 compared to original data: "depicted better separation between SAT and VAT with 6% increase in explanation of variation on PC1"
- [methods] Log2 transformation of both datasets prior to PCA comparison: "Both datasets were pre-processed as described previously [21,27] and log2-transformed prior to use in this study."
- [methods] lmm2met LMM fitting builds on lme4: "The package was implemented on a linear model-fitting procedure of the lme4 package"
- [readme] Install lmm2met from GitHub to run the tutorial: "Install lmm2met using the following commands"
