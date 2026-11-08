---
name: log-transformation-for-normality
description: Use when metabolomics abundance data (e.g. myocardial infarction, atherothrombotic,
  or African Race metabolite matrices) are skewed before KNN-TN imputation or distribution-based
  simulation of missing values.
when_to_use_negative:
- Metabolite values include zeros or negatives on the raw scale, so the logarithm
  is undefined unless an offset is first justified — the article does not describe
  an offset convention
- The downstream imputation method does not rely on a normality assumption (e.g. pure
  KNN-EU on Euclidean distance without truncated-normal standardization), where the
  transform is not required by the method
- Data are already approximately normal (e.g. previously log-processed intensities),
  where re-transforming adds no benefit
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: KNN-TN truncation-based imputation
  role: The downstream consumer of the log-transformed data; its truncated-normal
    MLE standardization (Newton-Raphson estimation of mean/sd) requires approximately
    normal metabolite distributions
- name: Newton-Raphson optimization
  role: Used after log-transformation to estimate truncated normal mean and standard
    deviation for metabolites near the LOD
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:f05d21f0b1f3dd1574d5d3491231aaa31d179723049229bfd59017353eb546ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KNN-TN truncation-based imputation
  - Newton-Raphson optimization
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Log-transformation-for-normality

## Summary

Log-transform metabolite abundance matrices so their distributions approximately satisfy normality before truncated-normal standardization and imputation (KNN-TN), since the KNN-TN likelihood and the gene-expression distribution simulation assume normal data.

## When to use

Use when metabolomics abundance data (e.g. myocardial infarction, atherothrombotic, or African Race metabolite matrices) are skewed before KNN-TN imputation or distribution-based simulation of missing values.

## When NOT to use

- Metabolite values include zeros or negatives on the raw scale, so the logarithm is undefined unless an offset is first justified — the article does not describe an offset convention
- The downstream imputation method does not rely on a normality assumption (e.g. pure KNN-EU on Euclidean distance without truncated-normal standardization), where the transform is not required by the method
- Data are already approximately normal (e.g. previously log-processed intensities), where re-transforming adds no benefit

## Inputs

- Raw metabolite abundance matrix (samples x metabolites) from mass spectrometry, with values on the original concentration scale
- Missingness mask / LOD threshold identifying below-detection-limit (MNAR) entries

## Outputs

- Log-transformed metabolite matrix with approximately normal per-metabolite distributions, ready for KNN-TN standardization and imputation
- Log-transformed complete matrix used as the base for generating 100 simulated replicate datasets

## How to apply

Apply the log transform to every metabolite column of the raw data matrix (e.g. the myocardial infarction 2x288, atherothrombotic 1032-metabolite, or African Race 1251/2250-metabolite matrices) before any imputation or simulation step. Rationale: the KNN-TN method estimates a truncated normal mean and standard deviation via Newton-Raphson for metabolites whose sample mean lies within three standard deviations of the LOD, and the gene-expression distribution simulation approach generates replicates from a normal model, so both require approximately normal inputs. After log-transforming, first impute any pre-existing missing values with KNN-CR (correlation distance d_C = 1 - |r|) so the simulation and RMSE evaluation operate on a complete reference matrix. The transform precedes, not replaces, the cleaning filter that removes metabolites with more than 75% missing observations.

## Related tools

- **KNN-TN truncation-based imputation** (The downstream consumer of the log-transformed data; its truncated-normal MLE standardization (Newton-Raphson estimation of mean/sd) requires approximately normal metabolite distributions)
- **Newton-Raphson optimization** (Used after log-transformation to estimate truncated normal mean and standard deviation for metabolites near the LOD)

## Evaluation signals

- Per-metabolite distributions of the transformed matrix are approximately normal (visual or moment check) before fitting the truncated normal mean and sd
- Newton-Raphson estimation of the truncated mean and standard deviation converges for metabolites whose sample mean lies within three standard deviations of the LOD
- After imputation, the expected RMSE ordering holds: mean RMSE for KNN-TN below KNN-CR, and both KNN-TN and KNN-CR below KNN-EU (p < 0.05 contrasts)
- The cleaning filter (metabolites with >75% missing observations removed) is applied after transformation and pre-imputation, matching the paper's pipeline order

## Limitations

- KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation, while metabolite data are often non-normal even after transformation
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN than KNN-CR

## Evidence

- [discussion] Log transform metabolites to approximately achieve normality prior to imputation: "we log transform them to approximately achieve normality prior to imputation"
- [intro] Truncated-normal imputation assumes MNAR+MAR missingness with LOD as truncation point: "we develop an imputation algorithm based on nearest neighbors that considers MNAR and MAR together based on a truncated distribution, with the detection limit considered as the"
- [abstract] KNN-TN improves imputation under MAR plus LOD missingness: "KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU when there is missingness due to missing at random combined with an"
