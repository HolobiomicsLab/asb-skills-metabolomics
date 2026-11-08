---
name: knn-truncation-imputation
description: Use when metabolomics or other high-dimensional mass-spectrometry data
  contain missing values arising both from metabolites below the detection limit (MNAR)
  and from random technical failures (MAR), and the analysis goal is to impute them
  accurately before downstream statistical testing (e.
when_to_use_negative:
- Data contain no meaningful detection-limit truncation (e.g. purely random missingness
  with no LOD) — the article notes KNN-TN was no better than KNN-EU imputation in
  one of the three real data sets, where the MNAR component is absent or weak.
- Metabolites have more than 75% missing values — these are removed in the cleaning
  process, not imputed.
- Sample sizes are very small or missingness excessive, making truncated normal mean/variance
  estimation unstable and inflating RMSE variability for KNN-TN relative to KNN-CR.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Newton-Raphson optimization
  role: Solves the likelihood equations for the truncated normal mean and standard
    deviation of each metabolite's observed values, supplying the standardization
    used inside the KNN-TN distance metric.
- name: KNN-TN truncation-based imputation
  role: 'The core imputation routine: correlation-distance KNN on truncated-normal-standardized
    data with sign-weighted averaging over K = 10 neighbors.'
- name: Gene expression distribution simulation approach
  role: Generates 100 replicate datasets per group from real metabolomics matrices
    to create controlled MNAR/MAR missingness for benchmarking KNN-TN against KNN-CR
    and KNN-EU.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:f80b6038dba3db3e3f9666f9d7777aea3ad8373bbff8f12f56ed9def3a25c2dd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gene expression distribution simulation approach
  - KNN-TN truncation-based imputation
  - Newton-Raphson optimization
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# knn-truncation-imputation

## Summary

A KNN imputation algorithm (KNN-TN) for high-dimensional metabolomics data that standardizes each metabolite against its truncated normal distribution — with the mass-spectrometer detection limit as the truncation point — to jointly account for MNAR (below-LOD) and MAR missingness, outperforming Euclidean- and correlation-distance KNN baselines.

## When to use

Use when metabolomics or other high-dimensional mass-spectrometry data contain missing values arising both from metabolites below the detection limit (MNAR) and from random technical failures (MAR), and the analysis goal is to impute them accurately before downstream statistical testing (e.g. RMSE evaluation or metabolite list concordance).

## When NOT to use

- Data contain no meaningful detection-limit truncation (e.g. purely random missingness with no LOD) — the article notes KNN-TN was no better than KNN-EU imputation in one of the three real data sets, where the MNAR component is absent or weak.
- Metabolites have more than 75% missing values — these are removed in the cleaning process, not imputed.
- Sample sizes are very small or missingness excessive, making truncated normal mean/variance estimation unstable and inflating RMSE variability for KNN-TN relative to KNN-CR.

## Inputs

- High-dimensional metabolomics abundance matrix (e.g. myocardial infarction: 2 groups × 288 metabolites; atherothrombotic: sCAD/Type1/Type2 × 1032 metabolites; African Race: Ethiopia 25×1251 and Tanzania 15×2250), log-transformed
- Detection limit (LOD) value for each metabolite, used as the truncation point
- Optional: simulated replicate matrices generated from real data with prescribed MNAR/MAR missingness patterns

## Outputs

- Completed data matrix with imputed missing values
- Per-metabolite truncated normal mean and standard deviation estimates (Newton–Raphson MLE)
- RMSE values comparing imputed entries to true values (mean and SD across replicates)
- Metabolite list concordance index (MLCI) assessing effect on downstream statistical testing

## How to apply

First log-transform metabolites to approximately achieve normality prior to imputation, and remove metabolites with more than 75% missing observations in the cleaning step. For each metabolite whose sample mean lies within three standard deviations of the detection limit, estimate the truncated normal mean and standard deviation via Newton–Raphson maximum likelihood on the observed values, then standardize the data accordingly; compute correlation distances d_C = 1 − |r| on this truncated-normal-standardized data. Select K = 10 nearest neighbors and impute missing entries by sign-weighted averaging with weights w_k = sign(r_k) · d_k⁻¹ / Σ_l d_l⁻¹, so negatively correlated neighbors pull imputed values downward — the key departure from KNN-CR and KNN-EU, which ignore the truncation. Evaluate by RMSE against known removed values and, for downstream impact, by the metabolite list concordance index (MLCI) after re-running differential testing.

## Related tools

- **Newton-Raphson optimization** (Solves the likelihood equations for the truncated normal mean and standard deviation of each metabolite's observed values, supplying the standardization used inside the KNN-TN distance metric.)
- **KNN-TN truncation-based imputation** (The core imputation routine: correlation-distance KNN on truncated-normal-standardized data with sign-weighted averaging over K = 10 neighbors.)
- **Gene expression distribution simulation approach** (Generates 100 replicate datasets per group from real metabolomics matrices to create controlled MNAR/MAR missingness for benchmarking KNN-TN against KNN-CR and KNN-EU.)

## Examples

```
import numpy as np; from scipy.optimize import root; from scipy.spatial.distance import cdist; X_log = np.log(X); mu, sd = truncated_mle(X_log, lod); D = cdist(((X_log - mu) / sd).T, ((X_log - mu) / sd).T, metric='correlation'); idx = np.argsort(D, axis=1)[:, :10]; imputed = weighted_average(idx, X_log)
```

## Evaluation signals

- KNN-TN achieves the lowest mean RMSE in every simulated condition, with KNN-TN vs KNN-EU contrasts significant at p < 0.05, matching the pattern in Tables 1–3.
- Per-metabolite truncated mean and SD estimates must be finite and within three standard deviations of the LOD threshold; metabolites outside this window fall back to other handling.
- In every case the mean RMSE for KNN-TN should be below that for KNN-CR, and both should be more than two standard deviations below KNN-EU's mean (p < 0.05).
- MLCI values for KNN-TN should be roughly equivalent to KNN-EU and superior to zero, mean, and minimum-value imputation in most cases.
- Imputed entries for below-LOD metabolites must remain below the truncation point, reflecting the MNAR mechanism.

## Limitations

- KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation, while metabolite data are often non-normal (mitigated by log-transformation).
- Estimation of truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN than KNN-CR.
- The KNN-TN likelihood is based solely on observed data; the missingness mechanism (MNAR, MAR, MCAR) is unknown and not incorporated into the likelihood.
- Downstream analysis evaluation (MLCI) is only preliminary; a more comprehensive study of all potential downstream analyses is still needed.
- No software, package, or code availability is stated in the paper despite the new algorithm being proposed.

## Evidence

- [intro] truncation-based KNN with detection limit: "In this work, we develop an imputation algorithm based on nearest neighbors that considers MNAR and MAR together based on a truncated distribution, with the detection limit considered as the"
- [abstract] KNN-TN outperforms other methods: "we show that KNN-TN has lower RMSE values compared to the other two KNN procedures as well as simpler imputation methods based on substituting missing values with the metabolite mean, zero values, or"
- [abstract] KNN-TN best under MNAR+LOD: "KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU when there is missingness due to missing at random combined with an"
- [discussion] log transform for normality: "we log transform them to approximately achieve normality prior to imputation"
- [discussion] cleaning removes >75% MVs: "in the cleaning process (which removes metabolites with >75% MVs)"
- [discussion] no better than KNN-EU in one dataset: "While the KNN-TN method was significantly better than four other imputation algorithms (zero imputation, minimum value imputation, and KNN-EU imputation) in two of three data sets, it was no better"
- [discussion] normality limitation: "A related limitation is the reliance on the normality assumption for estimating the truncated mean and standard deviation"
- [discussion] variability of RMSE higher for KNN-TN: "the variability of the RMSE for KNN-TN is higher than or similar to that for KNN-CR. This is directly related to the estimation of the mean and variance for the truncated normal distribution, which"
- [discussion] likelihood ignores missingness mechanism: "The likelihood used in our KNN-TN method is based solely on the observed metabolite data. The full data likelihood would include missing data as well. This is difficult to specify in the current"
- [discussion] no software stated: "we introduce an extension to the KNN imputation algorithm which handles truncated data, termed KNN-TN"
- [discussion] Tables 1–3 pattern: "the proposed method (KNN-TN) generally has lower RMSE values compared to the other two KNN methods and simpler imputation algorithms (zero, mean, and minimum value imputation)"
