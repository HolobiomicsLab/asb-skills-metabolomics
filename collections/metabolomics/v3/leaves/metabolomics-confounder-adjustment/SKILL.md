---
name: metabolomics-confounder-adjustment
description: Use when a raw metabolomics peak-intensity table (e.g., TEDDY Metabolomics
  Workbench ST001386) will feed network estimation or differential analysis, and sample
  covariates like age and sex must be removed from metabolite abundances first.
when_to_use_negative:
- Input is already normalized or covariate-adjusted (e.g., log-transformed, batch-corrected,
  or residualized) data — double adjustment would distort the signal
- The dataset lacks per-sample covariate metadata such as age and sex, so no regression
  adjustment is possible
- The downstream method requires strictly positive raw-scale concentrations rather
  than residual-shifted values (the min-shift to zero changes the intensity origin)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Fits per-metabolite linear models (intensity ~ age + sex) via lm() and computes
    the residual vectors that form the adjusted intensities
- name: DNEA R package
  role: 'Downstream consumer: the age/sex-adjusted, min-shifted intensity matrix is
    passed to createDNEAobject() as raw peak intensities for joint network estimation'
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:5ed69cba70e87ba585eddba874d60f3bbef9a720945bb0bc38c35ef81de59310
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNEA R package
  - Filigree
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolomics-confounder-adjustment

## Summary

Adjusts raw metabolomics peak-intensity values for demographic confounders (age, sex) by taking linear-regression residuals per metabolite and shifting them so each metabolite's minimum is zero, producing a covariate-corrected intensity matrix suitable for differential network construction in DNEA.

## When to use

Use when a raw metabolomics peak-intensity table (e.g., TEDDY Metabolomics Workbench ST001386) will feed network estimation or differential analysis, and sample covariates like age and sex must be removed from metabolite abundances first.

## When NOT to use

- Input is already normalized or covariate-adjusted (e.g., log-transformed, batch-corrected, or residualized) data — double adjustment would distort the signal
- The dataset lacks per-sample covariate metadata such as age and sex, so no regression adjustment is possible
- The downstream method requires strictly positive raw-scale concentrations rather than residual-shifted values (the min-shift to zero changes the intensity origin)

## Inputs

- Raw unnormalized metabolomics peak-intensity matrix (metabolites x samples), e.g., TEDDY Metabolomics Workbench ST001386 last-visit data
- Per-sample covariate metadata: age and sex
- Per-sample condition labels (case/control, e.g., IA case vs IA control) as a named factor vector
- Missingness-filtered and median-imputed metabolite table (metabolites missing in >30% of samples already excluded)

## Outputs

- Covariate-adjusted (age- and sex-corrected) peak-intensity matrix with each metabolite's minimum residual shifted to zero (all values >= 0)
- Adjusted feature table ready as input to createDNEAobject() or comparable network-construction routines

## How to apply

Start from unnormalized peak intensities for each metabolite across samples (after excluding individuals who switched condition, e.g., IA-control-to-IA-case converters, dropping metabolites missing in >30% of samples, and median-imputing remaining missing values). For each metabolite, fit a linear model with the metabolite intensity as the response and the confounders (age and sex in the TEDDY IA/T1D analysis) as predictors, then take the residuals. Shift each metabolite's residual vector by subtracting its minimum so all adjusted values are >= 0, preserving a non-negative intensity scale compatible with downstream tools like createDNEAobject(). Use the adjusted matrix, together with case/control group labels supplied as a named factor vector, as input to downstream joint network estimation; the paper applied this exact adjustment before DNEA runs on the IA and T1D TEDDY arms.

## Related tools

- **R** (Fits per-metabolite linear models (intensity ~ age + sex) via lm() and computes the residual vectors that form the adjusted intensities)
- **DNEA R package** (Downstream consumer: the age/sex-adjusted, min-shifted intensity matrix is passed to createDNEAobject() as raw peak intensities for joint network estimation) — http://www.github.com/Karnovsky-Lab/DNEA/

## Examples

```
fit <- lm(metabolite_intensity ~ age + sex, data = teddy); adj <- fit$residuals; adj_shifted <- adj - min(adj)
```

## Evaluation signals

- Every adjusted metabolite column has minimum exactly 0 (residuals shifted so each metabolite's minimum is zero) and contains no negative values
- Adjusted values correspond to lm residuals: regressing each adjusted metabolite on age and sex yields near-zero residual association with both covariates
- Metabolites missing in more than 30% of samples were excluded before imputation, and remaining missing values were replaced by per-metabolite medians prior to adjustment
- Downstream sanity check: the adjusted matrix plus a named case/control factor vector builds a valid DNEA object, and filtered edge counts (e.g., 172 total / 139 differential at |pcor| >= 0.2 for the BIC-tuned IA network) are reproducible
- Condition-conversion sanity check: individuals who converted from IA control to IA case were excluded before adjustment

## Limitations

- Residual shifting to a zero minimum assumes the downstream tool accepts raw-peak-intensity-scale, non-negative input; other pipelines may prefer unshifted residuals
- Adjustment only removes linear additive effects of the modeled covariates (age, sex); nonlinear or interaction confounding is not addressed
- The adjustment was applied to unnormalized peak intensities, so results depend on the upstream preprocessing choices (30% missingness cutoff, median imputation)

## Evidence

- [results] unnormalized peak intensity values adjusted for age and sex were used as input.: "unnormalized peak intensity values adjusted for age and sex were used as input."
- [methods] For the remaining metabolites, all missing values were imputed with the median value of each metabolite.: "For the remaining metabolites, all missing values were imputed with the median value of each metabolite."
- [readme] The package takes as input the raw peak intensity / concentrations for a set of features from two experimental conditions: "The package takes as input the raw peak intensity / concentrations for a set of features from two experimental conditions and uses the GLASSO algorithm"
