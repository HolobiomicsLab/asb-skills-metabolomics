---
name: truncated-normal-mle-estimation
description: Use when a metabolomics data matrix has missing values below a known
  limit of detection, and truncated-normal mean/SD estimates are needed as standardization
  parameters for KNN-TN imputation or distribution modeling.
when_to_use_negative:
- When missingness is purely MAR with no limit-of-detection component — the truncation
  model assumes MNAR below-LOD missingness combined with MAR
- When the metabolite's sample mean lies farther than three standard deviations from
  the LOD — per the paper's runtime rule, use the sample mean and SD directly instead
- When metabolite data are strongly non-normal — the method relies on the normality
  assumption for the truncated distribution, a stated limitation
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: Newton-Raphson optimization
  role: Iterative solver used to find the MLE of the truncated-normal mean and variance
    for each metabolite, initialized at the sample mean and sample SD
- name: KNN-TN truncation-based imputation
  role: Downstream consumer — the estimated truncated mean and SD serve as the standardization
    parameters for correlation-distance neighbor selection and sign-weighted averaging
    in KNN-TN
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:8a3abbd3913587934ccc156eaba799b69b1567d23e45adc62b622347f329f1d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Additional file 3 example script
  - KNN-TN truncation-based imputation
  - Newton-Raphson optimization
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# truncated-normal-mle-estimation

## Summary

Estimates the mean and standard deviation of each left-truncated normal metabolite distribution via maximum likelihood with Newton-Raphson optimization, using the limit of detection (LOD) as the truncation point. This provides the standardization parameters for the KNN-TN imputation algorithm when missingness combines MNAR (below-LOD) and MAR mechanisms.

## When to use

Use when a metabolomics data matrix has missing values below a known limit of detection, and truncated-normal mean/SD estimates are needed as standardization parameters for KNN-TN imputation or distribution modeling.

## When NOT to use

- When missingness is purely MAR with no limit-of-detection component — the truncation model assumes MNAR below-LOD missingness combined with MAR
- When the metabolite's sample mean lies farther than three standard deviations from the LOD — per the paper's runtime rule, use the sample mean and SD directly instead
- When metabolite data are strongly non-normal — the method relies on the normality assumption for the truncated distribution, a stated limitation

## Inputs

- Log-transformed metabolomics data matrix with missing values (e.g. myocardial infarction data with 288 metabolites, atherothrombotic data with 1032 metabolites, or African Race Ethiopia 25x1251 / Tanzania 15x2250 subsets)
- Known limit of detection (LOD) used as the left-truncation point a

## Outputs

- truncated_mle_estimates.csv containing the estimated truncated-normal mean and standard deviation for every metabolite, with flags for metabolites where Newton-Raphson failed to converge

## How to apply

Load the log-transformed metabolomics matrix and treat the LOD as the left-truncation point a. For each metabolite, omit missing values and write the log-likelihood of the left-truncated normal: l = -n*ln(P(Y in (a,inf)|mu,sigma^2)) - n*ln(sqrt(2*pi*sigma^2)) - sum((y_i-mu)^2)/(2*sigma^2). Initialize the Newton-Raphson optimizer with the observed-data sample mean and sample standard deviation, then iterate NR updates on (mu, sigma^2) until convergence. To accelerate run-time, apply truncated-normal estimation only to metabolites whose sample mean lies within three standard deviations of the LOD; for the remaining metabolites use the sample mean and SD directly. Flag any metabolite where NR fails to converge, which can occur when missingness below the LOD is too high.

## Related tools

- **Newton-Raphson optimization** (Iterative solver used to find the MLE of the truncated-normal mean and variance for each metabolite, initialized at the sample mean and sample SD)
- **KNN-TN truncation-based imputation** (Downstream consumer — the estimated truncated mean and SD serve as the standardization parameters for correlation-distance neighbor selection and sign-weighted averaging in KNN-TN)

## Examples

```
python estimate_truncated_mle.py --input metabolites_log.csv --lod 4.5 --output truncated_mle_estimates.csv
```

## Evaluation signals

- Newton-Raphson converges for all eligible metabolites (those with sample mean within three SDs of the LOD); non-convergence flags are empty or only present for metabolites with excessive below-LOD missingness
- All estimated truncated means and standard deviations are finite and usable as standardization parameters
- Estimated truncated SD should reflect left-truncation: for metabolites near the LOD, the truncated SD differs appropriately from the raw sample SD
- When fed forward into KNN-TN (K=10, distance d_C = 1 - |r| with sign(r_k)-weighted averaging), the resulting RMSE is lower than KNN-CR and KNN-EU under combined MNAR+MAR missingness

## Limitations

- Relies on the normality assumption for estimating the truncated mean and standard deviation, while metabolite data are often non-normal (mitigated in practice by log-transformation to approximately achieve normality prior to imputation)
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN than for KNN-CR
- NR may fail to converge when missingness below the LOD is too high
- The likelihood is based solely on the observed metabolite data; the missingness mechanism (MNAR, MAR, MCAR) is not incorporated into the likelihood

## Evidence

- [methods] NR initialization and eligibility rule: [evidence span withheld — anchor status: not_found]
- [methods] Runtime acceleration rule: [evidence span withheld — anchor status: not_found]
- [discussion] Normality and log-transform rationale: "A related limitation is the reliance on the normality assumption for estimating the truncated mean and standard deviation"
- [discussion] Variability limitation: "the variability of the RMSE for KNN-TN is higher than or similar to that for KNN-CR. This is directly related to the estimation of the mean and variance for the truncated normal distribution, which"
- [abstract] Truncation point definition: "we develop an imputation algorithm based on nearest neighbors that considers MNAR and MAR together based on a truncated distribution, with the detection limit considered as the truncation point"
