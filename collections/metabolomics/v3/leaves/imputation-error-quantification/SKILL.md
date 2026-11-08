---
name: imputation-error-quantification
description: Use when comparing missing-value imputation methods on complete benchmark
  data where values can be removed completely at random (10%, 20%, 30%) and true values
  are known, so NRMSE (continuous) and PFC (categorical) error rates can be computed
  per simulation.
when_to_use_negative:
- True values are unavailable for the missing entries (e.g. naturally occurring missingness),
  so NRMSE/PFC against ground truth cannot be computed — only OOB estimates remain,
  and the article treats these as an informal per-variable reliability check rather
  than formal error assessment
- Missingness is not completely at random; the evaluation framework here is built
  solely on artificially introduced MCAR values of 10% to 30%
- MissPALasso is being run on very high-dimensional data (e.g. the Insulin dataset,
  p=12,626), where the article states computation is not feasible
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: missForest
  role: Iterative random-forest imputation providing built-in out-of-bag imputation
    error estimates without a test set
  repo: http://stat.ethz.ch/CRAN/
- name: KNNimpute
  role: Baseline for continuous data (cross-validated k, standardized variables) and
    for categorical/mixed data after {-1,1} dummy encoding
- name: MissPALasso
  role: Baseline for continuous-only data on standardized variables; skipped for high-dimensional
    Insulin data
- name: MICE
  role: Baseline for categorical and mixed-type data; run on Children's Hospital data
    after removing observations with missing values and nearly dependent variables
- name: R
  role: Statistical environment hosting the missForest and MICE implementations and
    the paired Wilcoxon testing
  repo: http://stat.ethz.ch/CRAN/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btr597
    title: MissForest—non-parametric missing value imputation for mixed-type data
  source_content_hash: sha256:44959ffdb6f3f81b874f977c0fc80e3962a59ede17c53317067d0ae3facff3a8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - KNNimpute
  - MICE
  - MissPALasso
  - R
  - missForest
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# imputation-error-quantification

## Summary

Quantifies imputation accuracy of missForest against baselines (KNNimpute, MissPALasso, MICE, dummy-coded KNNimpute) using NRMSE for continuous variables and PFC for categorical variables over 50 simulations with 10–30% artificially removed values, plus random-forest out-of-bag error as a test-set-free error estimate.

## When to use

Use when comparing missing-value imputation methods on complete benchmark data where values can be removed completely at random (10%, 20%, 30%) and true values are known, so NRMSE (continuous) and PFC (categorical) error rates can be computed per simulation.

## When NOT to use

- True values are unavailable for the missing entries (e.g. naturally occurring missingness), so NRMSE/PFC against ground truth cannot be computed — only OOB estimates remain, and the article treats these as an informal per-variable reliability check rather than formal error assessment
- Missingness is not completely at random; the evaluation framework here is built solely on artificially introduced MCAR values of 10% to 30%
- MissPALasso is being run on very high-dimensional data (e.g. the Insulin dataset, p=12,626), where the article states computation is not feasible

## Inputs

- Complete data matrix (continuous-only, categorical-only, or mixed-type) with known true values, e.g. the ten benchmark datasets (Isoprenoid A. thaliana network p=39/n=118, Parkinson's voice data, Musk molecules, Insulin p=12,626/n=110, SPECT p=22 binary features, E. coli promoter sequences, Lymphography, Gaucher proteomics, C. elegans GFOP peptides, Children's Hospital n=55/p=124)
- Simulated incomplete matrix after removing values completely at random (10%, 20%, or 30%)
- Per-simulation error rates (NRMSE or PFC) for missForest and each compared method

## Outputs

- NRMSE values for continuous variables per simulation and missingness condition
- PFC values for categorical variables per simulation and missingness condition
- Out-of-bag imputation error estimates from missForest (per variable, informal reliability check)
- Paired Wilcoxon test p-values of compared method error rates versus missForest error rates
- Stratified relative error reductions by variable-type composition (e.g. missforest_gain_summary.csv-style summary table)

## How to apply

Introduce missing values completely at random at 10%, 20% and 30% into complete datasets (50 independent simulations; only 10 for the high-dimensional Insulin dataset, p=12,626), then impute with missForest and each setting-appropriate baseline: KNNimpute/MissPALasso for continuous-only data, MICE and dummy-encoded KNNimpute for categorical and mixed-type data. For continuous data, standardize variables to unit standard deviation and zero mean before KNNimpute and MissPALasso, and retransform imputed matrices to original scales before computing the normalized root mean squared error (NRMSE). For categorical data, code variables into {-1,1}-dummy variables for KNNimpute, impute, code dummies back, and score the proportion of falsely classified (PFC) entries. Compare paired error rates across the 50 simulations with a paired Wilcoxon test of the compared method's rates versus missForest's rates, and compute relative error reduction = (baseline error − missForest error)/baseline error. As a test-set-free check, report missForest's random-forest out-of-bag (OOB) imputation error estimates, which the authors found adequate in all settings.

## Related tools

- **missForest** (Iterative random-forest imputation providing built-in out-of-bag imputation error estimates without a test set) — http://stat.ethz.ch/CRAN/
- **KNNimpute** (Baseline for continuous data (cross-validated k, standardized variables) and for categorical/mixed data after {-1,1} dummy encoding)
- **MissPALasso** (Baseline for continuous-only data on standardized variables; skipped for high-dimensional Insulin data)
- **MICE** (Baseline for categorical and mixed-type data; run on Children's Hospital data after removing observations with missing values and nearly dependent variables)
- **R** (Statistical environment hosting the missForest and MICE implementations and the paired Wilcoxon testing) — http://stat.ethz.ch/CRAN/

## Examples

```
library(missForest); ximp <- missForest(xmis); ximp$OOBerror
```

## Evaluation signals

- Each continuous-variable result is reported as NRMSE computed on data retransformed to original scales after standardization-based imputation
- Each categorical-variable result is reported as PFC (proportion of falsely classified values), with dummy variables coded back to categorical variables before scoring
- Across conditions, missForest's NRMSE reductions of up to ~25% versus KNNimpute (and up to ~50–60% vs. baselines in some settings) are the expected order of magnitude
- Paired Wilcoxon tests over the 50 simulations yield significant differences in the directions reported per dataset; significance statements are withheld for MICE on Children's Hospital data due to ad-hoc manipulation (removal of missing-value rows and nearly dependent variables)
- OOB error estimates should track the true NRMSE/PFC per variable; only 10 (not 50) simulations are expected for the Insulin dataset

## Limitations

- Evaluation relies entirely on artificially introduced MCAR missingness of 10% to 30%; no results for non-random missingness mechanisms
- MissPALasso results are absent for the Insulin data set because high dimension makes computation infeasible
- Only 10 simulations instead of 50 were performed for the Insulin data set due to computation time
- MICE results on the Children's Hospital data required ad-hoc manipulation (removing observations with missing values and nearly dependent variables) and must be treated cautiously; no significance statements are reported there
- OOB imputation error estimates are an informal mean reliability check per variable, not a formal error assessment

## Evidence
