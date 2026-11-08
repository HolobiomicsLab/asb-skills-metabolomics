---
name: metabolomics-data-simulation-from-real-datasets
description: Use when the goal is to benchmark imputation algorithms on real metabolomics
  data by injecting a known missingness pattern (10% MNAR below the 10%-quantile LOD,
  5% MAR) into complete data and scoring downstream metabolite-list concordance (MLCI).
when_to_use_negative:
- When the goal is direct imputation-accuracy assessment on genuinely missing values
  (RMSE against observed values) rather than downstream list-concordance evaluation
  on simulated data
- When the source metabolomics data have no usable complete reference (e.g., metabolite
  prevalence is too low to survive the >75% missing-value filter), so a gold-standard
  metabolite list cannot be established
- When the missingness mechanism of interest differs from the article's 15% (10% MNAR
  below the 10%-quantile LOD + 5% MAR) design and the simulation must not impose it
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Metabolomics WorkBench
  role: Repository from which the African Race complete subsets (Project ID PR000010;
    Ethiopia 25x1251 and Tanzania 15x2250) are obtained as simulation source data
- name: Additional file 3 example script
  role: Supplementary example script accompanying the workflow for reproducing the
    simulation and MLCI comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:cea5c7e6c7a5ba5a40497c877677674f347c29dfa2912803afd8d9f7f04c1b72
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Additional file 3 example script
  - Metabolomics WorkBench
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabolomics-data-simulation-from-real-datasets

## Summary

Generates controlled simulated metabolomics datasets from real complete data (or KNN-CR-completed real data) so imputation methods can be scored on downstream statistical concordance (MLCI) against a known gold standard, using a missingness model of 10% MNAR below the LOD plus 5% MAR.

## When to use

Use when the goal is to benchmark imputation algorithms on real metabolomics data by injecting a known missingness pattern (10% MNAR below the 10%-quantile LOD, 5% MAR) into complete data and scoring downstream metabolite-list concordance (MLCI).

## When NOT to use

- When the goal is direct imputation-accuracy assessment on genuinely missing values (RMSE against observed values) rather than downstream list-concordance evaluation on simulated data
- When the source metabolomics data have no usable complete reference (e.g., metabolite prevalence is too low to survive the >75% missing-value filter), so a gold-standard metabolite list cannot be established
- When the missingness mechanism of interest differs from the article's 15% (10% MNAR below the 10%-quantile LOD + 5% MAR) design and the simulation must not impose it

## Inputs

- Real complete metabolomics dataset (e.g., myocardial infarction data, 288 metabolites; atherothrombotic data, 1032 metabolites at baseline; African Race data, Metabolomics WorkBench Project ID PR000010: Ethiopia 25x1251 and Tanzania 15x2250)
- For incomplete source datasets: missing-value data completed by KNN-CR imputation (K=10, metabolites standardized to mean zero and SD one)
- Gold-standard significant metabolite list from the complete data (reference for MLCI)
- Missingness specification: 10% MNAR below LOD (10% quantile of complete data) + 5% MAR; number of simulations (100 per dataset/group)

## Outputs

- 100 simulated datasets per dataset/group with 15% injected missingness and metabolites with >75% missing observations removed
- Per-simulation significant metabolite lists from two-group t-tests on mean log intensity
- Summary table of mean MLCI (with SDs) per dataset per imputation method, plus ANOVA contrast tests comparing methods (cf. Additional file 2: Tables S13-S15)

## How to apply

Start from real complete metabolite datasets: the myocardial infarction data (288 metabolites), the human atherothrombotic data (1032 metabolites, baseline time point), and the African Race complete subsets from Metabolomics WorkBench Project PR000010 (Ethiopia 25x1251, Tanzania 15x2250). For datasets with missing values (myocardial infarction, atherothrombotic), first complete them with KNN-CR (K=10, correlation-based neighbor selection with metabolites standardized to mean zero and SD one), then generate 100 simulated datasets per dataset/group using the Parmigiani et al. distribution-modeling approach; for the African Race data, simulate directly from the complete subsets. Log-transform metabolites to approximately achieve normality, then introduce 15% missingness per simulation as 10% MNAR below the LOD (defined as the 10% quantile of the complete data) plus 5% MAR placed randomly in the non-missing entries, and remove metabolites with more than 75% missing observations. Run two-group t-tests on mean log intensity for each simulated dataset, compute the MLCI of the significant metabolite list against the gold-standard list from the complete data, average MLCI (with SDs) over the 100 simulations, and test mean MLCI differences between methods via pre-defined ANOVA contrasts.

## Related tools

- **Metabolomics WorkBench** (Repository from which the African Race complete subsets (Project ID PR000010; Ethiopia 25x1251 and Tanzania 15x2250) are obtained as simulation source data)
- **Additional file 3 example script** (Supplementary example script accompanying the workflow for reproducing the simulation and MLCI comparison)

## Evaluation signals

- The averaged MLCI table must reproduce the Table 4 pattern, e.g., myocardial KNN-TN MLCI of 0.510 (0.097) and zero imputation MLCI of 0.061 (0.054)
- Across the three real datasets, KNN-TN (MLCI 0.510, 0.392, 0.274) and KNN-CR must have the highest MLCI values while zero, minimum, mean, and KNN-EU have lower MLCI indexes
- KNN-TN must be significantly better than zero, minimum, and KNN-EU imputation in two of three datasets, with no significant difference from mean imputation on the African Race data
- Simulated datasets must carry exactly 15% missingness (10% MNAR below the 10%-quantile LOD + 5% MAR) and no metabolite with more than 75% missing observations should remain

## Limitations

- KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation, while metabolite data are often non-normal
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN
- The KNN-TN likelihood is based solely on observed data; the missingness mechanism (MNAR, MAR, MCAR) is unknown and not incorporated
- Downstream analysis evaluation is only preliminary; a comprehensive study of all potential downstream analyses is still needed

## Evidence

- [abstract] missingness due to missing at random combined with a limit of detection (LOD): "missingness due to missing at random combined with an LOD"
- [intro] Account for both missingness mechanisms: "Since missingness may be due to a metabolite being below the detection limit of the mass spectrometer (MNAR) or other technical issues unrelated to the abundance of the metabolite (MAR)"
- [abstract] Downstream statistical testing evaluation via MLCI: "the metabolite list concordance index (MLCI) for influence on downstream statistical testing"
- [discussion] 75% missing-value filter: "in the cleaning process (which removes metabolites with >75% MVs)"
- [discussion] Log transform prior to imputation: "we log transform them to approximately achieve normality prior to imputation"
- [discussion] Limitation: normality assumption for truncated mean and SD estimation: "A related limitation is the reliance on the normality assumption for estimating the truncated mean and standard deviation"
