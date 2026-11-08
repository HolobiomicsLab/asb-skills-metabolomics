---
name: mnar-mar-missingness-injection
description: Decision support for benchmarking imputation methods on complete metabolomics
  matrices when you need missing values reflecting both MNAR (below-LOD) and MAR mechanisms,
  e.g. 15% missingness with more MNAR than MAR.
when_to_use_negative:
- The input matrix already contains missing values — the skill assumes a complete
  matrix so deletions create known ground truth; imputing an already-incomplete dataset
  needs a different workflow.
- Missingness in your data is entirely below-LOD (pure MNAR) or entirely random (pure
  MCAR) with no mixture — a single-mechanism mask is then more appropriate than the
  combined MNAR+MAR scheme.
- The goal is to impute real incomplete clinical/pre-clinical metabolomics data for
  downstream analysis rather than to evaluate an imputer against simulated ground
  truth.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Newton-Raphson optimization
  role: Not used by the injection step itself; listed in the source card's tool vocabulary,
    Newton–Raphson is used downstream by KNN-TN to fit truncated-normal means/SDs
    on the masked data, so the injected mask must preserve enough observed values
    for the optimizer to converge.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:761d208b33a7e8c280491a22d0760883a9f6062dea69aebf9b55cf76b5ef6a69
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Newton-Raphson optimization
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mnar-mar-missingness-injection

## Summary

Injects controlled missingness into complete metabolomics data by combining below-limit-of-detection (MNAR) deletions and random (MAR/MCAR) deletions, so that imputation algorithms such as KNN-TN, KNN-CR, and KNN-EU can be benchmarked against known ground truth.

## When to use

Decision support for benchmarking imputation methods on complete metabolomics matrices when you need missing values reflecting both MNAR (below-LOD) and MAR mechanisms, e.g. 15% missingness with more MNAR than MAR.

## When NOT to use

- The input matrix already contains missing values — the skill assumes a complete matrix so deletions create known ground truth; imputing an already-incomplete dataset needs a different workflow.
- Missingness in your data is entirely below-LOD (pure MNAR) or entirely random (pure MCAR) with no mixture — a single-mechanism mask is then more appropriate than the combined MNAR+MAR scheme.
- The goal is to impute real incomplete clinical/pre-clinical metabolomics data for downstream analysis rather than to evaluate an imputer against simulated ground truth.

## Inputs

- Complete (fully observed) log-transformed metabolomics data matrix — samples × metabolites (real e.g. myocardial infarction data, or simulated multivariate-normal data with Uniform(−5, 5) means)
- Chosen total missing fraction (e.g. 15%, 20%, 30%, 40%, 50%)
- MNAR fraction (values below LOD to delete, e.g. 10%)
- MAR fraction (values deleted at random, e.g. 5%)
- LOD definition, e.g. the 10% quantile of each complete metabolite distribution

## Outputs

- Incomplete data matrix with the specified mix of MNAR (below-LOD) and random (MAR) missing entries
- Binary missingness mask indicating which entries were removed by MNAR vs MAR deletion
- Complete ground-truth matrix retained for RMSE and MLCI evaluation
- Cleaned matrix after applying the >75% missing-value metabolite filter

## How to apply

Start from a fully observed, log-transformed metabolite matrix (e.g. simulated from a multivariate normal with means from Uniform(−5, 5) and one of the paper's blockwise, AR(1), or mixed correlation structures). Define the limit of detection (LOD) as a low quantile of the complete data — the paper uses the 10% quantile — and remove a chosen fraction of values that fall below it (these are MNAR), then remove an additional fraction completely at random (MAR/MCAR), e.g. 10% below LOD plus 5% random to reach 15% total missingness. Follow the paper's cleaning rule: drop metabolites with more than 75% missing observations so the injected mask is not dominated by hopelessly sparse rows. Vary the total missing percentage and the MNAR/MAR balance (as in the paper's 10–50% range) to generate evaluation sets, and retain the complete matrix as ground truth for computing RMSE and MLCI after imputation. The rationale is that metabolomics missingness is a mixture: values are either below the instrument's detection limit (MNAR) or lost to technical errors unrelated to abundance (MAR), and an imputation benchmark that only deletes values at random will fail to penalize methods that cannot handle truncated distributions.

## Related tools

- **Newton-Raphson optimization** (Not used by the injection step itself; listed in the source card's tool vocabulary, Newton–Raphson is used downstream by KNN-TN to fit truncated-normal means/SDs on the masked data, so the injected mask must preserve enough observed values for the optimizer to converge.)

## Examples

```
mask = np.random.rand(X.shape[0], X.shape[1]) < 0.05; lod = np.percentile(X, 10, axis=0); X_masked[X < lod] = np.nan  # 10% MNAR + 5% MAR = 15% total
```

## Evaluation signals

- Total fraction of removed entries equals the target missingness level (e.g. exactly 15% of sample×metabolite cells deleted).
- The MNAR-deleted cells all fall below the defined LOD (the 10% quantile of the complete data), and the remaining deletions are uniformly random (MAR).
- No metabolite exceeds the paper's cleaning threshold of >75% missing observations after injection; any that do are removed before imputation.
- The complete ground-truth matrix is retained, allowing exact RMSE computation between imputed and true values.
- Downstream testing (e.g. t-tests / ANOVA on imputed data) can be compared to tests on the complete data via the metabolite list concordance index (MLCI), giving a second check that the mask is realistic.

## Limitations

- The paper's evaluation covers sample sizes and missing percentages only above certain thresholds; performance 'unless the missing percentage is below the values evaluated in this study (data not shown)' — injection-based benchmarks are unvalidated for very small samples (<20) or very low missingness.
- Results depend on how the LOD is chosen (10% quantile here); a different LOD changes the MNAR/MAR balance and therefore the ranking of imputers such as KNN-TN vs KNN-EU.
- Simulated data assume multivariate-normal metabolite distributions after log transform; real metabolomics data are often non-normal, so injected masks on such data may not reproduce the paper's findings.

## Evidence

- [abstract] missing values (MVs) in metabolomics datasets are common and can arise due to both technical and biological reasons: "missing values (MVs) in metabolomics datasets are common and can arise due to both technical and biological reasons"
- [intro] In metabolomics studies, we assume that the missing values occurs either as MNAR (metabolites occur at low abundances, below the detection limit) or MAR: "In metabolomics studies, we assume that the missing values occurs either as MNAR (metabolites occur at low abundances, below the detection limit) or MAR"
- [abstract] missingness due to missing at random combined with an LOD: "missingness due to missing at random combined with an LOD"
- [discussion] in the cleaning process (which removes metabolites with >75% MVs): "in the cleaning process (which removes metabolites with >75% MVs)"
- [discussion] we log transform them to approximately achieve normality prior to imputation: "we log transform them to approximately achieve normality prior to imputation"
