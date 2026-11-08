---
name: metabolomics-log-transformation-scaling
description: Use when preparing targeted LC/MS or GC/MS metabolomics matrices (e.g.,
  real_data.csv, targeted_data.csv) with non-normal variable distributions and missing
  values, before running GS_impute or comparable imputation methods.
when_to_use_negative:
- When variables are already approximately normal on the raw scale, log-transformation
  (applied 'for non-normal data') may be unnecessary and could distort the distribution.
- When the goal is imputation of right-censored missing values, the pipeline should
  use 'rsym' initialization with an informative lower truncation point instead of
  the QRILC-initialized left-censored setup.
- When the data are not a metabolomics intensity matrix (e.g., already a feature table
  or sample metadata), scaling/recovery semantics of the wrapper do not apply.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GSimp
  role: Provides the pre_processing_GS_wrapper, scale_recover, and GS_impute functions
    implementing the log-transform / QRILC-initialize / scale / impute / recover pipeline
  repo: https://github.com/WandeRum/GSimp
- name: imputeLCMD (QRILC)
  role: impute.QRILC performs the initial fill of missing values on the log-transformed
    matrix before scaling
  repo: https://github.com/WandeRum/GSimp
- name: R
  role: Language in which the pre-processing and imputation pipeline is implemented
    and distributed
  repo: https://github.com/WandeRum/GSimp
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:295eb84ef85c1b1d11a65ff109f1409c848ef43b2a129c57a7d72e31294922ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - QRILC
  - R
  - glmnet
  - imputeLCMD (QRILC)
  - kNN-TN
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# metabolomics-log-transformation-scaling

## Summary

Pre-processing pipeline for targeted metabolomics data prior to GSimp imputation: log-transformation of non-normal data, QRILC initialization of missing values, centralization/scaling for elastic-net prediction, followed by scaling and exponential recovery of the imputed matrix. It is wrapped in the pre_processing_GS_wrapper function for one-step processing and imputation.

## When to use

Use when preparing targeted LC/MS or GC/MS metabolomics matrices (e.g., real_data.csv, targeted_data.csv) with non-normal variable distributions and missing values, before running GS_impute or comparable imputation methods.

## When NOT to use

- When variables are already approximately normal on the raw scale, log-transformation (applied 'for non-normal data') may be unnecessary and could distort the distribution.
- When the goal is imputation of right-censored missing values, the pipeline should use 'rsym' initialization with an informative lower truncation point instead of the QRILC-initialized left-censored setup.
- When the data are not a metabolomics intensity matrix (e.g., already a feature table or sample metadata), scaling/recovery semantics of the wrapper do not apply.

## Inputs

- Raw targeted metabolomics data matrix with missing values (e.g., real_data.csv containing FFAs and BAs, targeted_data.csv, untargeted_data.csv)
- Vector of NA positions from the original data (NA_pos)

## Outputs

- Log-scaled, QRILC-initialized data matrix with NAs restored at original missing positions (input to GS_impute)
- Scaling parameter data frame (data_raw_log_qrilc_sc_df_param) for later recovery
- Final imputed data matrix recovered to the original intensity scale (data_imp)

## How to apply

Log-transform the raw data with log() since metabolomics intensities are typically non-normal; then initialize missing positions with impute.QRILC on the log-scale matrix. Apply centralization and scaling via scale_recover(data_raw_log_qrilc, method='scale'), which also returns the scaling parameters needed later for recovery. Reintroduce NAs at the original missing positions (NA_pos from which(is.na(data_raw), arr.ind=T)) so the initialized-but-scaled matrix can be fed to GS_impute with initial=data_raw_log_qrilc_sc_df, lo=-Inf, hi='min'. After imputation, invert the transformations with scale_recover(., method='recover', param_df=...) and exp() to return values on the original intensity scale.

## Related tools

- **GSimp** (Provides the pre_processing_GS_wrapper, scale_recover, and GS_impute functions implementing the log-transform / QRILC-initialize / scale / impute / recover pipeline) — https://github.com/WandeRum/GSimp
- **imputeLCMD (QRILC)** (impute.QRILC performs the initial fill of missing values on the log-transformed matrix before scaling) — https://github.com/WandeRum/GSimp
- **R** (Language in which the pre-processing and imputation pipeline is implemented and distributed) — https://github.com/WandeRum/GSimp

## Examples

```
source('GSimp.R'); data_imp <- pre_processing_GS_wrapper(real_data)
```

## Evaluation signals

- The imputed output must be back on the original intensity scale: values after scale_recover(..., method='recover') and exp() should match the magnitude of non-missing entries in the raw input matrix.
- Only positions recorded in NA_pos (which(is.na(data_raw), arr.ind=T)) should carry imputed values; all originally observed entries must be unchanged.
- Scaling parameters (data_raw_log_qrilc_sc_df_param) returned by the first scale_recover call must be reused for recovery — mismatched parameters will shift the recovered distribution.
- Scaled log matrix fed to GS_impute should have NAs exactly at the original missing positions (data_raw_log_sc[NA_pos] <- NA), not QRILC-initialized values.
- Re-imputing the same input with identical arguments (iters_each=50, iters_all=10, lo=-Inf, hi='min') should reproduce NRMSE/SOR results consistent with the paper's evaluation, where GSimp attains the lowest Sum of Ranks.

## Limitations

- The paper does not specify version numbers for key R packages (glmnet, vegan, ropls, imputeLCMD) beyond those listed in the README table (e.g., imputeLCMD 2, glmnet 2.0-13), so exact reproducibility of the QRILC initialization may depend on package versions.
- Scaling is performed 'for elastic-net prediction'; the pipeline is tied to GSimp's glmnet_pred model and may not be appropriate for imputers that do not assume a scaled log-space.
- Log-transformation requires strictly positive values; zeros or negative intensities in the raw matrix would produce NaNs.

## Evidence

- [readme] In GSimp, we recommend data pre-processing steps as following: "In GSimp, we recommend data pre-processing steps as following: -   Log-transformation (for non-normal data) -   Initialization for missing values (e.g., QRILC) -   Centralization and scaling (for"
- [readme] Wrapper for one-step processing: "All aboved steps has been wrapped into the *pre\_processing\_GS\_wrapper* function for a one-step processing and imputation."
- [readme] Recovery steps: "data_imp <- data_imp_log_sc %>% 
    scale_recover(., method = 'recover', 
                  param_df = data_raw_log_qrilc_sc_df_param) %>% 
    extract2(1) %>% exp()"
- [methods] Article methods on pre-processing: "-   Log-transformation (for non-normal data) -   Initialization for missing values (e.g., QRILC)"
