---
name: simulated-metabolomics-data-generation
description: Use when benchmarking or validating imputation algorithms on metabolomics
  data and ground-truth values are needed to compute RMSE/MLCI, or when a controlled
  MNAR+LOD/MAR missingness setting must be reproduced.
when_to_use_negative:
- The input is real experimental metabolomics data where true missing values are unknown
  and RMSE against ground truth cannot be computed
- The target data are strongly non-normal, since the downstream KNN-TN likelihood
  relies on a normality assumption for the truncated mean and SD
- The goal is to study the full missing-data likelihood including the (unknown) missingness
  mechanism, which the truncated-observation likelihood does not model
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: Newton-Raphson optimization
  role: After simulation, fits the truncated-normal MLE of the mean and SD (with sample
    estimates as starting values) that KNN-TN uses on the simulated data
- name: KNN-TN
  role: 'Consumer of the simulated dataset: truncation-based KNN imputer benchmarked
    against KNN-CR and KNN-EU using the simulated ground truth'
- name: KNN-CR
  role: Correlation-distance (d_C = 1 − |r|) KNN imputer applied to the simulated
    incomplete matrix as a comparator
- name: KNN-EU
  role: Euclidean-distance KNN imputer applied to the simulated incomplete matrix
    as a comparator
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:9e89c211015afeadd6439aa2c63031b427f005893b81757b770f331ee6f9ce46
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KNN-CR
  - KNN-EU
  - KNN-TN
  - Newton-Raphson optimization
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# simulated-metabolomics-data-generation

## Summary

Generates synthetic metabolomics matrices from a multivariate normal distribution with controlled correlation structures and controlled missingness (MNAR below an LOD plus MAR), so that imputation methods such as KNN-TN, KNN-CR, and KNN-EU can be benchmarked against known true values.

## When to use

Use when benchmarking or validating imputation algorithms on metabolomics data and ground-truth values are needed to compute RMSE/MLCI, or when a controlled MNAR+LOD/MAR missingness setting must be reproduced.

## When NOT to use

- The input is real experimental metabolomics data where true missing values are unknown and RMSE against ground truth cannot be computed
- The target data are strongly non-normal, since the downstream KNN-TN likelihood relies on a normality assumption for the truncated mean and SD
- The goal is to study the full missing-data likelihood including the (unknown) missingness mechanism, which the truncated-observation likelihood does not model

## Inputs

- Sample size N (e.g., 50) and metabolite count M (e.g., 400)
- Metabolite mean distribution specification (Uniform(−5, 5))
- Correlation structure choice: blockwise (ρw=0.7, ρoff=0.2), AR(1) (ρ=0.9), or mixed blockwise
- Total missingness percentage and its MNAR/MAR split (e.g., 15% = 10% below LOD + 5% MAR)
- LOD definition (quantile of the complete data, e.g., 10% quantile)

## Outputs

- Complete simulated metabolomics matrix (samples × metabolites) as ground truth
- Incomplete matrix with imposed MNAR-below-LOD and MAR missingness
- Cleaned dataset after removing metabolites with >75% missing values
- Per-metabolite truncated-normal parameters (mean, SD) usable by KNN-TN

## How to apply

Simulate an N-sample × M-metabolite dataset (e.g., 50 samples × 400 metabolites) from a multivariate normal distribution with metabolite means drawn from Uniform(−5, 5) and one of the paper's correlation structures: blockwise with ρw=0.7 within blocks and ρoff=0.2 between blocks, AR(1) with ρ=0.9, or a mixed blockwise structure. Impose a total missingness fraction (e.g., 15%) by splitting it between MNAR — removing a percentage of values below the LOD, defined as a quantile of the complete data (e.g., the 10% quantile) — and MAR values removed at random (e.g., 5%). Log transform the simulated metabolites to approximately achieve normality prior to imputation, then apply the cleaning step that removes metabolites with more than 75% missing observations. Keep the complete-data matrix as ground truth so RMSE and the metabolite list concordance index (MLCI) can be computed after imputation with K=10 neighbors.

## Related tools

- **Newton-Raphson optimization** (After simulation, fits the truncated-normal MLE of the mean and SD (with sample estimates as starting values) that KNN-TN uses on the simulated data)
- **KNN-TN** (Consumer of the simulated dataset: truncation-based KNN imputer benchmarked against KNN-CR and KNN-EU using the simulated ground truth)
- **KNN-CR** (Correlation-distance (d_C = 1 − |r|) KNN imputer applied to the simulated incomplete matrix as a comparator)
- **KNN-EU** (Euclidean-distance KNN imputer applied to the simulated incomplete matrix as a comparator)

## Examples

```
mu = np.random.uniform(-5, 5, 400); X = np.random.multivariate_normal(mu, Sigma, size=50)  # 50x400 simulated metabolomics matrix
```

## Evaluation signals

- Simulated matrix has the requested dimensions (e.g., 50 samples × 400 metabolites) and metabolite means fall within Uniform(−5, 5)
- Empirical correlations of the simulated data match the chosen structure (blockwise ρw≈0.7 within blocks and ρoff≈0.2 between blocks, or AR(1) ρ≈0.9)
- Observed missingness fraction equals the requested total (e.g., 15%) with the requested MNAR/MAR split (e.g., 10% of values below the 10%-quantile LOD plus 5% MAR)
- Cleaning step removes exactly the metabolites with more than 75% missing observations
- Ground-truth complete matrix is retained so that imputation RMSE and MLCI can be recomputed and compared to the paper's reported patterns (e.g., KNN-TN lower RMSE than KNN-CR/KNN-EU)

## Limitations

- KNN-TN's estimation of the truncated mean and SD relies on a normality assumption, while metabolite data are often non-normal
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability
- Results for sample sizes under 20 and missing percentages below those evaluated were referenced but not shown ('data not shown')

## Evidence

- [intro] In this work, we develop an imputation algorithm based on nearest neighbors that considers MNAR and MAR together based on a truncated distribution: "In this work, we develop an imputation algorithm based on nearest neighbors that considers MNAR and MAR together based on a truncated distribution"
- [discussion] cleaning process removes metabolites with >75% MVs: "in the cleaning process (which removes metabolites with >75% MVs)"
- [abstract] missing values arise from technical and biological reasons: "missing values (MVs) in metabolomics datasets are common and can arise due to both technical and biological reasons"
