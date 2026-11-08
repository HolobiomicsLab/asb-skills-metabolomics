---
name: metabolomics-sample-normalization-factor-calculation
description: Use when you have a feature intensity table (features in rows, samples
  in columns) from untargeted metabolomics and need per-sample normalization factors
  that reflect relative sample concentrations, reducing intragroup variation.
when_to_use_negative:
- The data lack serial QC samples and high-quality feature selection/intensity correction
  is required first — MDFC fold-change accuracy depends on MAFFIN's upstream FeatureSelection
  and IntCorrection modules
- The goal is comparison against alternative normalizations (Sum, Median, Quantile,
  PQN) rather than computing MDFC factors — use the corresponding SumNorm/MedianNorm/QuantileNorm/PQNNorm
  functions instead
- The input is raw spectral data rather than a feature intensity table — MAFFIN expects
  an already peak-picked feature table
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MAFFIN
  role: R package implementing MDFC normalization via MDFCNorm and factor application
    via NormByFactors
  repo: https://github.com/HuanLab/MAFFIN
- name: MDFCNorm
  role: Function that computes per-sample MDFC normalization factors using Gaussian
    kernel density estimation with bwOpt SJ bandwidth option
  repo: https://github.com/HuanLab/MAFFIN
- name: NormByFactors
  role: Applies the computed normalization factors to the feature table and can run
    the RMAD evaluation
  repo: https://github.com/HuanLab/MAFFIN
- name: EvaPRMAD
  role: Evaluates normalization quality by computing pooled RMAD / pooled RSD of the
    normalized table
  repo: https://github.com/HuanLab/MAFFIN
- name: R
  role: Runtime environment; MAFFIN is installed via devtools::install_github
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac355
    title: 'MAFFIN: metabolomics sample normalization using maximal density fold change
      with high-quality metabolic features and corrected signal intensities'
  source_content_hash: sha256:2c57135b67401f2688855ea4113bb5b46b12bc0bd88cccefba8d8fb324dc456d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/HuanLab/MAFFIN
  repo_url: https://github.com/HuanLab/MAFFIN
  tools:
  - EvaPRMAD
  - MAFFIN
  - MDFCNorm
  - NormByFactors
  - R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# metabolomics-sample-normalization-factor-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calculates metabolomics sample normalization factors via maximal density fold change (MDFC): per-feature log2 fold changes against a reference sample are smoothed with a Gaussian kernel density estimator, and the fold change at maximal density becomes the normalization factor.

## When to use

Use when you have a feature intensity table (features in rows, samples in columns) from untargeted metabolomics and need per-sample normalization factors that reflect relative sample concentrations, reducing intragroup variation.

## When NOT to use

- The data lack serial QC samples and high-quality feature selection/intensity correction is required first — MDFC fold-change accuracy depends on MAFFIN's upstream FeatureSelection and IntCorrection modules
- The goal is comparison against alternative normalizations (Sum, Median, Quantile, PQN) rather than computing MDFC factors — use the corresponding SumNorm/MedianNorm/QuantileNorm/PQNNorm functions instead
- The input is raw spectral data rather than a feature intensity table — MAFFIN expects an already peak-picked feature table

## Inputs

- Feature intensity table (data frame / .csv) with features in rows and samples in columns, first row containing sample group names (blank, RT, QC, SQC_### labels supported)
- IntThreshold detection criterion value
- bwOpt bandwidth option (Sheather-Jones rule for two-sample tables)
- SampleInCol flag (features-in-rows, samples-in-columns layout)

## Outputs

- Named numeric vector of MDFC normalization factors per sample (written to mdfc_norm_factors.csv)
- Normalized feature intensity table when combined with NormByFactors
- Pooled RMAD (PRMAD) intragroup variation metric for quality control when RunEvaluation is enabled

## How to apply

Load the feature intensity table with group labels in the first row (blank, RT, QC, SQC_### for serial QC), and identify the sample with the most detected features as the reference. Compute per-feature log2 fold changes of every sample's intensities against the reference, applying the IntThreshold detection criterion as in MDFCNorm. Estimate the density of the fold changes with a Gaussian kernel density estimator, using the Sheather-Jones (SJ) bandwidth rule when the table contains only two samples (bwOpt parameter of MDFCNorm). Take the fold change at the maximal kernel density as each sample's normalization factor, assemble the named numeric vector, and optionally evaluate quality with pooled RMAD (EvaPRMAD) before writing factors to mdfc_norm_factors.csv.

## Related tools

- **MAFFIN** (R package implementing MDFC normalization via MDFCNorm and factor application via NormByFactors) — https://github.com/HuanLab/MAFFIN
- **MDFCNorm** (Function that computes per-sample MDFC normalization factors using Gaussian kernel density estimation with bwOpt SJ bandwidth option) — https://github.com/HuanLab/MAFFIN
- **NormByFactors** (Applies the computed normalization factors to the feature table and can run the RMAD evaluation) — https://github.com/HuanLab/MAFFIN
- **EvaPRMAD** (Evaluates normalization quality by computing pooled RMAD / pooled RSD of the normalized table) — https://github.com/HuanLab/MAFFIN
- **R** (Runtime environment; MAFFIN is installed via devtools::install_github)

## Examples

```
NormFactors = MDFCNorm(FeatureTable, IntThreshold, SampleInCol=TRUE, output=FALSE, OutputNormFactors=TRUE, RunEvaluation=TRUE, bwOpt)
```

## Evaluation signals

- Recovered normalization factor from a synthetic two-sample table with a known dilution factor of 1.5 matches the expected dilution relationship (built-in self-test)
- Pooled RMAD of the normalized table is lower than the original (pre-normalization) RMAD, indicating reduced intragroup variation
- Output NormFactor is a named numeric vector with one factor per biological sample
- MDFC outperforms four commonly used normalization methods (Sum, Median, Quantile, PQN) in reducing intragroup variation on comparable input

## Limitations

- Fold-change accuracy depends on upstream high-quality feature selection and MS signal intensity correction using serial quality control samples; without serial QC data the correction module cannot run
- The SJ bandwidth rule (bwOpt) is specified for the two-sample case; behavior with other bandwidth settings must be verified
- Feature selection criteria (Blank, RT, LowRSD, LowCor) may exclude features, biasing the density estimate if too few high-quality features remain

## Evidence

- [methods] Kernel density estimate at maximum: [evidence span withheld — anchor status: not_found]
