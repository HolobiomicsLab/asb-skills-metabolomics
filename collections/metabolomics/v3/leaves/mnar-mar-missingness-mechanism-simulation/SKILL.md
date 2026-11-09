---
name: mnar-mar-missingness-mechanism-simulation
description: Decision support for comparing imputation methods when missing values
  may arise from MNAR (metabolite below detection limit) and/or MAR (technical issues),
  and controlled simulations are needed to separate mechanism effects.
when_to_use_negative:
- When the dataset is real and the actual missingness mechanism can be estimated from
  observed quantile detection limits rather than simulated
- When only one missingness mechanism is plausible (pure MNAR or pure MAR without
  a 2:1 mix), since the benchmark design assumes both mechanisms are present
- When sample size or missingness percentage falls below the evaluated ranges (e.g.,
  under 20 samples or below 9% missing), as the paper notes results for such cases
  were only referenced as 'data not shown'
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Newton-Raphson optimization
  role: Used within KNN-TN to estimate truncated-normal mean and SD (LOD as truncation
    point) during imputation of the simulated data
- name: Multi-factor ANOVA
  role: Statistical testing of RMSE values with main effects for method, percent missing,
    and correlation type, plus pre-defined method contrasts
- name: R (runtime-implemented simulation and imputation scripts)
  role: Runtime environment for generating simulated datasets, applying KNN imputations,
    and running ANOVA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:f15077a33996165708959f2d29b7c180842fc65a7a748667373429805c26a1f5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Multi-factor ANOVA
  - Newton-Raphson optimization
  - R (runtime-implemented simulation and imputation scripts)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mnar-mar-missingness-mechanism-simulation

## Summary

A simulation skill for generating metabolomics-style high-dimensional datasets with controlled missingness mechanisms (MNAR below a limit-of-detection and MAR), used to benchmark imputation methods such as KNN-TN, KNN-CR, and KNN-EU via RMSE and ANOVA.

## When to use

Decision support for comparing imputation methods when missing values may arise from MNAR (metabolite below detection limit) and/or MAR (technical issues), and controlled simulations are needed to separate mechanism effects.

## When NOT to use

- When the dataset is real and the actual missingness mechanism can be estimated from observed quantile detection limits rather than simulated
- When only one missingness mechanism is plausible (pure MNAR or pure MAR without a 2:1 mix), since the benchmark design assumes both mechanisms are present
- When sample size or missingness percentage falls below the evaluated ranges (e.g., under 20 samples or below 9% missing), as the paper notes results for such cases were only referenced as 'data not shown'

## Inputs

- Simulation configuration: number of samples (20, 50, 100), number of metabolites (400, 900), correlation structure (blockwise positive, AR(1), blockwise mixed)
- Missingness specification: target percent missing (9%, 15%, 30%), MNAR-to-MAR ratio (2:1), quantile LOD defining MNAR values
- Number of replications (100) and random seed

## Outputs

- Simulated complete and missing-modified data matrices per replication and configuration
- Per-cell imputed vs. true RMSE values across replications, methods, and configurations
- Boxplots of RMSE distributions by method, correlation type, and percent missing (mirroring Figures 3-5)
- Multi-factor ANOVA tables with pre-defined contrasts (KNN-TN vs KNN-CR, KNN-TN vs KNN-EU) and p-values

## How to apply

Draw replicate data from an M-dimensional multivariate normal with metabolite means sampled Uniform(-5, 5) under three correlation structures (blockwise rho_w = 0.7 / rho_off = 0.2, AR(1) rho = 0.9, blockwise mixed positive/negative) at dimensions 20x400, 50x400, and 100x900. Introduce 9%, 15%, and 30% missingness per dataset with a 2:1 MNAR-to-MAR split, where MNAR values are defined as observations below the quantile LOD and MAR values are removed at random. Apply the cleaning rule that removes metabolites with more than 75% missing observations before imputation. Evaluate competing imputations (e.g., KNN-TN, KNN-CR, KNN-EU with K = 10 and inverse-distance weights) by computing RMSE against true values across 100 replications, then test method contrasts with multi-factor ANOVA (main effects: method, percent missing, correlation type).

## Related tools

- **Newton-Raphson optimization** (Used within KNN-TN to estimate truncated-normal mean and SD (LOD as truncation point) during imputation of the simulated data)
- **Multi-factor ANOVA** (Statistical testing of RMSE values with main effects for method, percent missing, and correlation type, plus pre-defined method contrasts)
- **R (runtime-implemented simulation and imputation scripts)** (Runtime environment for generating simulated datasets, applying KNN imputations, and running ANOVA)

## Examples

```
Rscript knn_tn_simulation.R --samples 50 --metabolites 400 --corr ar1 --missing 0.15 --replications 100
```

## Evaluation signals

- KNN-TN should show consistently lower mean RMSE than KNN-CR and KNN-EU across all configurations
- ANOVA contrasts KNN-TN vs KNN-CR and KNN-TN vs KNN-EU should yield statistically significant differences (p < 0.05) for all three sample-size cases
- Missingness split must be enforced at 2:1 MNAR-to-MAR with MNAR defined by the quantile LOD
- Metabolites exceeding 75% missing observations must be removed by the cleaning step before imputation
- RMSE boxplots should mirror the qualitative pattern of the paper's Figures 3-5

## Limitations

- KNN-TN relies on a normality assumption for estimating truncated mean and SD, while metabolite data are often non-normal
- Estimation of truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN
- The likelihood used is based solely on observed data; the true missingness mechanism (MNAR, MAR, MCAR) is unknown and not incorporated
- Results for sample sizes under 20 and missing percentages below those evaluated are referenced but not shown ('data not shown')

## Evidence

- [results] Across the simulated datasets, KNN-TN consistently achieved lower RMSE than both KNN-CR and KNN-EU, and ANOVA modeling of the RMSE values showed statistically significant differences between KNN-TN and the other two KNN methods for all three sample-size cases.: [evidence span withheld — anchor status: not_found]
- [intro] Two examples of metabolite distributions which have missing values (MVs), from the myocardial infarction data.: "Two examples of metabolite distributions which have missing values (MVs), from the myocardial infarction data"
- [intro] In metabolomics studies, we assume that the missing values occurs either as MNAR (metabolites occur at low abundances, below the detection limit) or MAR, e.g., metabolites are truly not present or: "In metabolomics studies, we assume that the missing values occurs either as MNAR (metabolites occur at low abundances, below the detection limit) or MAR"
- [abstract] KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU when there is missingness due to missing at random combined with an LOD.: "KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU"
- [discussion] in the cleaning process (which removes metabolites with >75% MVs): "in the cleaning process (which removes metabolites with >75% MVs)"
