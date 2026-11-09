---
name: covariate-adjustment-via-linear-regression-residuals
description: Use when raw metabolomics peak intensities (e.g., Metabolomics Workbench
  ST001386/ST001636 TEDDY data) contain demographic covariates such as age and sex
  that confound a case/control contrast, before creating the DNEA object.
when_to_use_negative:
- The feature table has already been normalized, residualized, or otherwise adjusted
  for age and sex — double adjustment would distort the data
- The dataset has no demographic or technical covariates confounding the group comparison
  (then pass intensities directly to createDNEAobject() which log-transforms and auto-scales
  each group)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Language in which the per-metabolite linear models are fit and residuals extracted
    (e.g., lm() / residuals()) prior to DNEA object creation
- name: DNEA R package
  role: Consumes the residual-adjusted expression matrix via createDNEAobject(), which
    further log-transforms and auto-scales each experimental group
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:4062e8d35e140bf774be878626c82c76adb056d863be3c7bc1bd2eb77c0711c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape
  - DNEA R package
  - R
  - glasso
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# covariate-adjustment-via-linear-regression-residuals

## Summary

Pre-processing step that removes confounding effects of age and gender from metabolomics peak-intensity data by regressing each metabolite on the covariates and keeping the residuals, so downstream DNEA joint network estimation reflects case-vs-control biology rather than demographic structure.

## When to use

Use when raw metabolomics peak intensities (e.g., Metabolomics Workbench ST001386/ST001636 TEDDY data) contain demographic covariates such as age and sex that confound a case/control contrast, before creating the DNEA object.

## When NOT to use

- The feature table has already been normalized, residualized, or otherwise adjusted for age and sex — double adjustment would distort the data
- The dataset has no demographic or technical covariates confounding the group comparison (then pass intensities directly to createDNEAobject() which log-transforms and auto-scales each group)

## Inputs

- Unnormalized peak-intensity metabolomics feature table (metabolites x samples), e.g., TEDDY data from Metabolomics Workbench ST001386/ST001636
- Sample metadata table (T1Dmeta) containing per-subject age, gender/sex, and case/control labels (e.g., 'DM:case' vs 'DM:control')

## Outputs

- Covariate-adjusted metabolite expression matrix (residuals per metabolite) aligned to the retained samples, ready for createDNEAobject()

## How to apply

After QC filtering (drop metabolites missing in >30% of samples, median-impute remaining missing values, remove individuals who converted from control to case), fit a linear regression of each metabolite's unnormalized peak intensity on the covariates (age, sex/gender) using the sample metadata (T1Dmeta), and replace each metabolite's values with the regression residuals. The residuals, not the original intensities, are then used as input to createDNEAobject(), which log-transforms and auto-scales each group ('DM:control' vs 'DM:case') independently. This is done before network estimation because DNEA's glasso/GGM model would otherwise encode covariate-driven correlations as edges. Do not additionally scale the residuals yourself — the DNEA object creation handles per-group scaling.

## Related tools

- **R** (Language in which the per-metabolite linear models are fit and residuals extracted (e.g., lm() / residuals()) prior to DNEA object creation)
- **DNEA R package** (Consumes the residual-adjusted expression matrix via createDNEAobject(), which further log-transforms and auto-scales each experimental group) — http://www.github.com/Karnovsky-Lab/DNEA/

## Examples

```
adj <- t(apply(exprData, 1, function(y) residuals(lm(y ~ T1Dmeta$age + T1Dmeta$gender))))
```

## Evaluation signals

- After adjustment, each metabolite's residual vector should be approximately mean-zero across samples, with no systematic age or gender trend when regressed again
- The case/control labels ('DM:control' vs 'DM:case') must be preserved unchanged; only expression values are residualized, and converter individuals must have been dropped before adjustment
- Downstream validation: the resulting DNEA networks should contain 290 total edges (70 control-specific, 112 case-specific, 108 shared), confirming the adjustment did not destroy group signal
- createDNEAobject() should pass its eigenvalue/condition-number stability check on the residualized data, indicating the matrix is suitable for GGM estimation

## Limitations

- Linear regression residuals only remove linear additive effects of age and sex; nonlinear confounding or covariate-by-group interactions are not addressed
- Adjustment is coupled to the DNEA pipeline's subsequent per-group log-transform and auto-scaling; residuals should not be independently z-scored before createDNEAobject()

## Evidence

- [other] adjust for age and gender by taking residuals of a linear regression model: [evidence span withheld — anchor status: not_found]
- [results] unnormalized peak intensity values adjusted for age and sex were used as input.: "unnormalized peak intensity values adjusted for age and sex were used as input. DNEA analysis was performed on IA and T1D datasets using all samples"
