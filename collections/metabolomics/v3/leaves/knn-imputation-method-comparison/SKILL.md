---
name: knn-imputation-method-comparison
description: Decision support for choosing among KNN imputation algorithms when a
  metabolomics dataset has missing values (MVs) arising from both MNAR (below LOD)
  and MAR mechanisms, and RMSE/MLCI comparisons are needed.
when_to_use_negative:
- When the dataset has no missing values or missingness is purely MAR/MCAR with no
  LOD component — the truncation-based machinery of KNN-TN offers no advantage and
  the article shows simpler methods can be equivalent (e.g. 'on the African Race data
  set there was no significant difference between any of the KNN imputation algorithms
  and mean imputation').
- When metabolites have >75% missing values — the cleaning process removes them before
  imputation, so benchmarking KNN variants on them is inappropriate.
- When the user needs a non-KNN or non-truncated-distribution approach; this skill
  only covers the three KNN procedures described in the paper.
edam_operation: http://edamontology.org/operation_3554
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3365
tools:
- name: KNN-TN
  role: 'truncation-based KNN imputation: estimates truncated normal mean and SD via
    Newton-Raphson MLE with the LOD as truncation point, then uses those estimates
    for nearest-neighbor imputation'
- name: KNN-CR
  role: 'correlation-based KNN imputation: selects neighbors using a correlation-distance
    metric among samples'
- name: KNN-EU
  role: 'Euclidean-based KNN imputation: selects neighbors using a Euclidean-distance
    metric among samples'
- name: Newton-Raphson optimization for truncated normal MLE
  role: numerical routine used inside KNN-TN to estimate the truncated mean and standard
    deviation from the observed (above-LOD) data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:9e3a54e168c92d1ea048a89ee3f49074210ddbc24011f0f7367212d83d935e69
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KNN-CR
  - KNN-CR (correlation-based KNN imputation)
  - KNN-EU
  - KNN-EU (Euclidean-based KNN imputation)
  - KNN-TN
  - KNN-TN (truncation-based KNN imputation)
  - Newton-Raphson optimization for truncated normal MLE
  edam_operation: http://edamontology.org/operation_3554
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# knn-imputation-method-comparison

## Summary

A comparative evaluation skill for metabolomics missing-value imputation that benchmarks truncation-based KNN (KNN-TN) against correlation-based (KNN-CR) and Euclidean-based (KNN-EU) KNN procedures using RMSE and the metabolite list concordance index (MLCI) on simulated and real datasets.

## When to use

Decision support for choosing among KNN imputation algorithms when a metabolomics dataset has missing values (MVs) arising from both MNAR (below LOD) and MAR mechanisms, and RMSE/MLCI comparisons are needed.

## When NOT to use

- When the dataset has no missing values or missingness is purely MAR/MCAR with no LOD component — the truncation-based machinery of KNN-TN offers no advantage and the article shows simpler methods can be equivalent (e.g. 'on the African Race data set there was no significant difference between any of the KNN imputation algorithms and mean imputation').
- When metabolites have >75% missing values — the cleaning process removes them before imputation, so benchmarking KNN variants on them is inappropriate.
- When the user needs a non-KNN or non-truncated-distribution approach; this skill only covers the three KNN procedures described in the paper.

## Inputs

- metabolomics abundance matrix (samples × metabolites) with missing values (MVs)
- simulated dataset (e.g. 50 × 400 from a multivariate normal) with imposed 10% MNAR-below-LOD + 5% MAR missingness
- limit of detection (LOD) threshold defining the truncation point
- three real metabolomics datasets used in the paper, e.g. myocardial infarction data [15] and African Race data set

## Outputs

- per-method RMSE values (KNN-TN vs KNN-CR vs KNN-EU vs zero/mean/minimum-value imputation)
- metabolite list concordance index (MLCI) for downstream statistical testing
- visual scatter/dot plot (Fig. 6 style) showing observed, true missing, and imputed values per metabolite with LOD shading
- concordance-in-significance-testing assessment across imputation methods

## How to apply

Generate or obtain a metabolomics dataset (e.g. 50 samples × 400 metabolites from a multivariate normal distribution) and impose 15% missingness composed of 10% MNAR below the limit of detection (LOD) and 5% MAR; log transform metabolites to approximately achieve normality prior to imputation. Run three KNN imputations with k=10: KNN-TN, which estimates the truncated normal mean and standard deviation via Newton-Raphson optimization of the truncated-normal likelihood with the LOD as the truncation point; KNN-CR, which uses a correlation-based distance metric; and KNN-EU, which uses a Euclidean-distance metric. Compute RMSE against the true missing values and, for downstream evaluation, the metabolite list concordance index (MLCI) from the downstream statistical testing results. Visual comparison (e.g. Fig. 6 style) restricts to the first 20 metabolites, plotting observed values as open black circles, true missing values as black stars, and imputed values as KNN-TN blue triangles, KNN-CR red squares, and KNN-EU green diamonds, with the region below the LOD shaded in light red.

## Related tools

- **KNN-TN** (truncation-based KNN imputation: estimates truncated normal mean and SD via Newton-Raphson MLE with the LOD as truncation point, then uses those estimates for nearest-neighbor imputation)
- **KNN-CR** (correlation-based KNN imputation: selects neighbors using a correlation-distance metric among samples)
- **KNN-EU** (Euclidean-based KNN imputation: selects neighbors using a Euclidean-distance metric among samples)
- **Newton-Raphson optimization for truncated normal MLE** (numerical routine used inside KNN-TN to estimate the truncated mean and standard deviation from the observed (above-LOD) data)

## Evaluation signals

- KNN-TN should yield lower RMSE than KNN-CR, KNN-EU, zero, mean, and minimum-value imputation when missingness is MAR combined with an LOD, particularly for values below the LOD.
- In Fig. 6-style single-dataset plots (N = 50, M = 400, 15% missing with 10% below LOD), KNN-TN's imputed values (blue triangles) should fall closer to the true missing values (black stars) than KNN-CR (red squares) and KNN-EU (green diamonds) for below-LOD metabolites such as metabolites 1, 3, 4, 7, 8, 12, and 13.
- MLCI values for KNN-TN and KNN-EU should be roughly equivalent and superior to the other four methods in most cases.
- Concordance in statistical significance testing: KNN-TN and KNN-CR should be roughly equivalent and generally outperform zero, mean, and minimum-value imputation.
- Cleaning step invariant: metabolites with >75% missing values must have been removed before imputation; the imputation matrix contains no metabolite with missingness above that threshold.

## Limitations

- KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation, while metabolite data are often non-normal (log transform is applied to approximately achieve normality prior to imputation).
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN compared to KNN-CR.
- The likelihood used in KNN-TN is based solely on the observed metabolite data; the full likelihood including missing data is difficult to specify and the missingness mechanism (MNAR, MAR, MCAR) is not incorporated.
- Results for sample sizes under 20 and missing percentages below those evaluated are referenced but not shown ('data not shown').
- No software, package, or code availability is stated anywhere in the paper despite the proposal of KNN-TN, so reproducibility requires re-implementation.
- Downstream analysis evaluation is only preliminary; a more comprehensive study of all potential downstream analyses is needed to fully determine whether improved imputation accuracy translates into better downstream results.

## Evidence

- [abstract] KNN-TN has lower RMSE values compared to KNN-CR, KNN-EU, and simpler imputation methods (metabolite mean, zero values, LOD): "we show that KNN-TN has lower RMSE values compared to the other two KNN procedures as well as simpler imputation methods based on substituting missing values with the metabolite mean, zero values, or"
- [abstract] KNN-TN generally improves imputation performance when missingness is due to MAR combined with a limit of detection: "KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU when there is missingness due to missing at random combined with an"
- [discussion] Log transform metabolites to approximately achieve normality prior to imputation: "we log transform them to approximately achieve normality prior to imputation"
- [discussion] Cleaning process removes metabolites with >75% missing values: "in the cleaning process (which removes metabolites with >75% MVs)"
- [discussion] KNN-TN and KNN-CR were roughly equivalent in concordance of statistical significance testing and outperformed the other four methods: "Assessment based on concordance in statistical significance testing demonstrate that KNN-TN and KNN-CR are roughly equivalent and generally outperform the other four methods"
- [discussion] Limitation: KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation: "A related limitation is the reliance on the normality assumption for estimating the truncated mean and standard deviation"
- [discussion] Limitation: estimation of truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN: "the variability of the RMSE for KNN-TN is higher than or similar to that for KNN-CR. This is directly related to the estimation of the mean and variance for the truncated normal distribution"
- [discussion] Limitation: the KNN-TN likelihood is based solely on observed data; the missingness mechanism is unknown and not incorporated: "The likelihood used in our KNN-TN method is based solely on the observed metabolite data. The full data likelihood would include missing data as well. This is difficult to specify in the current"
- [discussion] No software, package, or code availability is stated anywhere in the paper despite proposing a new algorithm (KNN-TN): "we introduce an extension to the KNN imputation algorithm which handles truncated data, termed KNN-TN"
- [discussion] Downstream analysis evaluation is only preliminary; a comprehensive study of all potential downstream analyses is still needed: "a more comprehensive study of all potential downstream analyses is needed to fully determine, whether the improved imputation accuracy of the KNN-TN method translates into better downstream"
- [full_text] Fig. 6 workflow: generate 50×400 simulated dataset with 15% missingness (10% below LOD, 5% MAR), impute with KNN-TN/KNN-CR/KNN-EU using k=10, plot first 20 metabolites with marker styles and LOD shading: [evidence span withheld — anchor status: not_found]
- [full_text] Fig. 6 finding: KNN-TN was in most cases able to impute missing values below the LOD better than KNN-CR and KNN-EU: [evidence span withheld — anchor status: not_found]
