---
name: multi-assay-lcms-data-integration
description: Use when two or more LC-MS assay feature tables (e.g. HPOS, LPOS, LNEG
  from the same cohort) must be integrated to model a binary or continuous outcome
  such as biological sex or blood bilirubin levels.
when_to_use_negative:
- Only a single assay feature table is available — fit a single-block PLS model instead
  of an MB-PLS integration
- The goal is structural clustering or annotation of already-selected significant
  features — use MamsiStructSearch rather than re-running the MB-PLS integration
- Data are raw LC-MS instrument files (mzML/NetCDF) rather than preprocessed feature
  tables — feature extraction must occur first
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: mamsi
  role: Python framework providing the MamsiPls class (NIPALS MB-PLS, estimate_lv,
    montecarlo_cv, kfold_cv, mb_vip, mb_vip_permtest, calculate_ci, block_importance)
  repo: https://github.com/kopeckylukas/py-mamsi
- name: mbpls 1.0.4
  role: Underlying multiblock PLS dependency extended by MamsiPls
- name: pandas
  role: Loading assay CSV feature tables and metadata, adding assay prefixes, and
    encoding the numeric outcome
- name: scikit-learn
  role: Train/test splitting of the data into aligned subsets before model fitting
- name: numpy
  role: Masking features by p-value threshold for selection of statistically significant
    features
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:853af4752978a58c8414deb370ca4ab5348a3700bce6846d2bc9d99b6ca07834
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - Python
  - mamsi
  - matplotlib
  - mbpls
  - mbpls 1.0.4
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# multi-assay-lcms-data-integration

## Summary

Integrates multiple multi-assay LC-MS metabolomics feature tables into a single multiblock partial least-squares (MB-PLS) model using the MAMSI Python framework, enabling prediction of a phenotypic outcome and quantification of each data block's contribution.

## When to use

Use when two or more LC-MS assay feature tables (e.g. HPOS, LPOS, LNEG from the same cohort) must be integrated to model a binary or continuous outcome such as biological sex or blood bilirubin levels.

## When NOT to use

- Only a single assay feature table is available — fit a single-block PLS model instead of an MB-PLS integration
- The goal is structural clustering or annotation of already-selected significant features — use MamsiStructSearch rather than re-running the MB-PLS integration
- Data are raw LC-MS instrument files (mzML/NetCDF) rather than preprocessed feature tables — feature extraction must occur first

## Inputs

- Multi-assay LC-MS feature tables as CSV (e.g. alz_hpos.csv, alz_lpos.csv, alz_lneg.csv from the AddNeuroMed cohort, or assay tables from the MY Diabetes cohort)
- Sample metadata CSV with outcome variable (e.g. alz_metadata.csv containing Gender labels or bilirubin values)
- Outcome variable encoded numerically (binary 0/1 for classification; continuous values for regression)

## Outputs

- Fitted MB-PLS model with block scores T_, super scores Ts_, superweights W_, and block importances A_
- Estimated number of latent variables from cross-validated LV estimation
- Per-repeat MCCV/k-fold evaluation metrics (accuracy, AUC, F1, precision, recall, specificity for classification; Q2, RMSE for regression)
- MB-VIP scores and empirical per-feature p-values
- MB-PLS vs single-block PLS performance comparison table (e.g. mccv_performance_comparison.csv)

## How to apply

Load each assay feature table (e.g. alz_hpos.csv, alz_lpos.csv, alz_lneg.csv) with pandas and add assay-specific prefixes to columns so features are interpretable downstream. Encode the outcome numerically (Gender as 1 for Female, 0 for Male) since the PLS algorithm requires a numeric response, then instantiate MamsiPls (NIPALS algorithm, standardise=True) and fit the model on the concatenated blocks, obtaining block scores T_, super scores Ts_, superweights W_ and block importances A_. Select the number of latent variables with MamsiPls.estimate_lv() using k-fold cross-validation (up to max_components=10, metric='auc' for classification); in the AddNeuroMed example this yields 6 latent variables. Evaluate predictive performance with MamsiPls.montecarlo_cv() (1000 repeats, test_size=0.2) or kfold_cv(), and compare against single-block PLS references to confirm that modeling interblock relationships improves outcome estimation. For feature-level interpretation, compute MB-VIP scores with mb_vip() and empirical p-values with mb_vip_permtest(), selecting features with p < 0.01.

## Related tools

- **mamsi** (Python framework providing the MamsiPls class (NIPALS MB-PLS, estimate_lv, montecarlo_cv, kfold_cv, mb_vip, mb_vip_permtest, calculate_ci, block_importance)) — https://github.com/kopeckylukas/py-mamsi
- **mbpls 1.0.4** (Underlying multiblock PLS dependency extended by MamsiPls)
- **pandas** (Loading assay CSV feature tables and metadata, adding assay prefixes, and encoding the numeric outcome)
- **scikit-learn** (Train/test splitting of the data into aligned subsets before model fitting)
- **numpy** (Masking features by p-value threshold for selection of statistically significant features)

## Examples

```
from mamsi.mamsi_pls import MamsiPls; mamsipls = MamsiPls(n_components=1); mamsipls.fit([hpos, lpos, lneg], y); mamsipls.estimate_lv([hpos, lpos, lneg], y, metric='auc')
```

## Evaluation signals

- The LV-estimation cross-validation curve for the AddNeuroMed classification task should plateau at 6 latent variables
- Mean MCCV validation AUC (1000 repeats, test_size=0.2) should be consistent with the cross-validated LV-estimate performance
- The final 6-LV model should reach held-out test metrics of accuracy 0.966, recall 1.0, specificity 0.943, F1 0.971, AUC 0.933
- MB-PLS minus single-block deltas should match reported values: Q2 +0.157 and RMSE −0.506 for bilirubin regression; accuracy/AUC/F1 deltas +0.055/+0.054/+0.061 for serum sex prediction
- All sample blocks must stay index-aligned after train/test splitting (subset the other two assays by the index of the first block)

## Limitations

- Requires preprocessed, sample-aligned feature tables per assay; blocks must be subset by the same sample indices to remain aligned
- Permutation testing with n_permutations=10000 is computationally expensive — HPC sample code (permtest_hpc) is provided for cluster execution
- The framework was tested on metabolomics phenotyping data; other LC-MS data types should be usable but are not guaranteed

## Evidence

- [intro] MAMSI is a Python framework designed for the integration of multi-assay mass spectrometry datasets.: "MAMSI is a Python framework designed for the integration of multi-assay mass spectrometry datasets."
- [abstract] MB-PLS outperformed single-assay models: "The MB-PLS models outperformed single-assay models in both classification and regression tasks, indicating that modeling interblock relationships enabled an improved estimate of phenotypic outcome."
- [methods] 6 latent variables: "The LV estimation result shows that the model has 6 latent variables/components."
- [readme] README quickstart data loading: "metadata = pd.read_csv('./sample_data/alz_metadata.csv')"
- [readme] framework tested on metabolomics phenotyping data: "*N.B. the framework was tested on metabolomics phenotyping data, but it should be usable with other types of LC-MS data.*"
