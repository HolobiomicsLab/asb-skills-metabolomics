---
name: sum-of-ranks-method-comparison
description: Use when comparing two or more imputation configurations (e.g., lo=-Inf/hi='min'
  vs LOQ- or 10%-quantile upper bounds) across a gradient of missingness levels and
  a single robust aggregate ranking is needed.
when_to_use_negative:
- Only a single missingness level is available — SOR is an aggregation across a gradient
  (e.g., seq(.1, .8, .1)) and is not meaningful for one level; report NRMSE alone
  instead
- There is no ground-truth mask (no simulated missingness generation), so NRMSE on
  imputed elements cannot be computed — use distribution- or analysis-based evaluation
  instead
- Comparing methods on fully observed data or on the real-world non-degraded matrix,
  where no imputed-element error is known
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: GSimp
  role: Provides GS_impute, MNAR_gen_imp, and the evaluation pipeline used to generate
    the per-configuration NRMSE values that the SOR aggregates
  repo: https://github.com/WandeRum/GSimp
- name: R
  role: Implementation language for rank aggregation and the NRMSE/SOR comparison
    table
  repo: https://github.com/WandeRum/GSimp
- name: QRILC
  role: impute.QRILC initialization used inside pre_processing_GS_wrapper so every
    compared configuration starts from the same initialized values
  repo: https://github.com/WandeRum/GSimp
- name: glmnet
  role: Elastic-net prediction model (imp_model='glmnet_pred') embedded in the Gibbs
    sampler for all compared imputation configurations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:8dca97c29044e1c61d5f70d08abf92569ff5caad3f759708a2032edcf8e2fe23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - QRILC
  - R
  - glmnet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Sum-of-ranks method comparison

## Summary

An imputation-benchmark aggregation that computes NRMSE on the log-scale imputed missing elements of every configuration at each missingness level and sums the per-level ranks (SOR) into a single comparison score, as used to rank GSimp truncation-point settings in targeted metabolomics.

## When to use

Use when comparing two or more imputation configurations (e.g., lo=-Inf/hi='min' vs LOQ- or 10%-quantile upper bounds) across a gradient of missingness levels and a single robust aggregate ranking is needed.

## When NOT to use

- Only a single missingness level is available — SOR is an aggregation across a gradient (e.g., seq(.1, .8, .1)) and is not meaningful for one level; report NRMSE alone instead
- There is no ground-truth mask (no simulated missingness generation), so NRMSE on imputed elements cannot be computed — use distribution- or analysis-based evaluation instead
- Comparing methods on fully observed data or on the real-world non-degraded matrix, where no imputed-element error is known

## Inputs

- Degraded MNAR data matrices (e.g., from targeted_data.csv, targeted LC/MS FFAs and BAs) at multiple missingness levels
- Imputed matrices from each configuration (default lo=-Inf/hi='min', LOQ bound, 10% quantile bound hi=sapply(data, function(x) quantile(x, .1, na.rm=T)), right-censored 'rsym', MCAR/MAR +/-Inf bounds)
- Positions of the originally missing elements in each degraded matrix

## Outputs

- Per-configuration, per-level NRMSE values computed on log-scale imputed missing elements
- Aggregated sum-of-ranks (SOR) scores across missingness levels
- Comparison table (e.g., truncation_comparison_results.csv) reporting NRMSE and SOR for each truncation configuration at each missingness level

## How to apply

Degrade the data to MNAR at graded levels (e.g., mis_var_prop=seq(.1, .8, .1), var_mis_prop=seq(.3, .6, .1) with cut-offs drawn from U(0.3, 0.6)), impute each degraded matrix with each configuration using GS_impute (e.g., iters_each=50, iters_all=10, imp_model='glmnet_pred'), then compute NRMSE on the log-scale values of the imputed missing elements only — not on observed entries. For each missingness level, rank the candidate configurations by NRMSE (lower NRMSE = better rank), and sum those ranks across the increasing missingness levels to obtain the SOR. Report NRMSE and SOR for each configuration at each level (e.g., into truncation_comparison_results.csv); because the source paper defines no numeric acceptance threshold, judge a relaxed bound (LOQ or 10% quantile) acceptable when its SOR is comparable to — not severely worse than — the default minimum-value bound.

## Related tools

- **GSimp** (Provides GS_impute, MNAR_gen_imp, and the evaluation pipeline used to generate the per-configuration NRMSE values that the SOR aggregates) — https://github.com/WandeRum/GSimp
- **R** (Implementation language for rank aggregation and the NRMSE/SOR comparison table) — https://github.com/WandeRum/GSimp
- **QRILC** (impute.QRILC initialization used inside pre_processing_GS_wrapper so every compared configuration starts from the same initialized values) — https://github.com/WandeRum/GSimp
- **glmnet** (Elastic-net prediction model (imp_model='glmnet_pred') embedded in the Gibbs sampler for all compared imputation configurations)

## Examples

```
R: `sor <- colSums(apply(nrmse_mat, 1, rank)) # rank configs within each missingness level, sum across levels
```

## Evaluation signals

- NRMSE is computed only on the log-scale values of the imputed missing elements, never on observed entries
- The comparison table reports both NRMSE and SOR for each configuration at each missingness level (e.g., levels .1 through .8)
- A relaxed bound (LOQ or 10% quantile) is judged acceptable when its SOR is comparable to, not severely worse than, the default hi='min' configuration
- All configurations are imputed under identical pre-processing (log-transformation, QRILC initialization, centralization/scaling) and identical Gibbs settings (iters_each=50, iters_all=10, imp_model='glmnet_pred') so ranks are attributable to the truncation bounds
- All imputed matrices are exported as CSVs alongside the comparison table so the ranking is reproducible from the raw outputs

## Limitations

- The paper defines no numeric acceptance threshold for SOR; the check is qualitative ('comparable to, not severely worse than')
- Rank-based aggregation discards NRMSE magnitude differences and can be sensitive to rank ties across levels
- Authors state truncation points should be set 'for different conditions' and that exact threshold/quantile settings for such conditions are not defined (e.g., 10% quantile or known LOQ/LOD values)

## Evidence

- [other] Compute NRMSE on the log-scale values of the imputed missing elements for every configuration and aggregate them into a sum-of-ranks (SOR) across the increasing missingness levels.: [evidence span withheld — anchor status: not_found]
- [other] MNAR degradation settings: [evidence span withheld — anchor status: not_found]
- [readme] Default truncation bounds in GS_impute: "Here, lo=-Inf, hi=‘min’ are default setting for
    left-censored missing values where the upper bound is set to the
    minimum value of non-missing part"
- [readme] Quantile bound example: "quantile values can be
    applied if minimum is too strict. For example, hi=sapply(data,
    function(x) quantile(x, .1, na.rm=T))"
- [readme] MVI-evaluation pipeline provenance: [evidence span withheld — anchor status: not_found]
