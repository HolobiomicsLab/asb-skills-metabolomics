---
name: proteomics-intensity-normalization
description: Use when a published metabolomics or proteomics feature intensity table
  lacks sufficient real QC samples and you need signal correction/normalization with
  simulated QC data, or when benchmarking linear vs nonlinear regression for correction.
when_to_use_negative:
- The dataset already contains a full set of measured QC samples, so real QC-based
  correction can be applied directly without simulating pseudo QC data
- The goal is raw data processing (peak detection, mzML/FASTQ handling) rather than
  regression-based correction of an already-computed feature intensity table
- The software environment is not R-based and the pseudoQC web interface or R code
  cannot be deployed
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: pseudoQC
  role: Web-based R software used to simulate QC sample data with four machine learning-based
    regression methods and apply them for correction and normalization of the datasets
  repo: https://github.com/qade544/pseudoQC
- name: R
  role: Implementation language for pseudoQC; environment in which the correction
    and normalization workflow is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201900264
    title: 'pseudoQC: A Regression‐Based Simulation Software for Correction and Normalization
      of Complex Metabolomics and Proteomics Datasets'
  source_content_hash: sha256:ebe2546f0e7efe4f96e688efa4e29b4b9e5b73be4200bf8cf0e763f527cf2d2f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/qade544/pseudoQC
  tools:
  - R
  - pseudoQC
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# proteomics-intensity-normalization

## Summary

Simulate QC sample data for metabolomics and proteomics feature-intensity tables using pseudoQC's four machine learning-based regression methods (two linear, two nonlinear), then apply the simulated QC data to correct and normalize the real datasets and compare regression classes by performance metrics.

## When to use

Use when a published metabolomics or proteomics feature intensity table lacks sufficient real QC samples and you need signal correction/normalization with simulated QC data, or when benchmarking linear vs nonlinear regression for correction.

## When NOT to use

- The dataset already contains a full set of measured QC samples, so real QC-based correction can be applied directly without simulating pseudo QC data
- The goal is raw data processing (peak detection, mzML/FASTQ handling) rather than regression-based correction of an already-computed feature intensity table
- The software environment is not R-based and the pseudoQC web interface or R code cannot be deployed

## Inputs

- Published metabolomics feature intensity table
- Published proteomics feature intensity table
- pseudoQC operating procedure (pseudoQC.Supplementary.pdf)

## Outputs

- Simulated QC sample data per dataset and per regression method
- Corrected/normalized feature intensity tables
- Per-method, per-dataset correction/normalization performance metrics (e.g. correction_normalization_comparison.csv)

## How to apply

Load the published metabolomics/proteomics feature intensity tables into the R-based pseudoQC environment following the operating procedure in pseudoQC.Supplementary.pdf. Simulate QC sample data for each dataset using the four machine learning-based regression methods implemented in pseudoQC (two linear and two nonlinear regressors). Use each simulated QC set to perform signal correction and normalization of the corresponding real dataset, computing per-method, per-dataset metrics such as residual signal drift and relative standard deviation of QC/internal-standard features. Compare metrics across methods; the paper's expected result is that nonlinear regression methods perform better than linear ones for both published datasets.

## Related tools

- **pseudoQC** (Web-based R software used to simulate QC sample data with four machine learning-based regression methods and apply them for correction and normalization of the datasets) — https://github.com/qade544/pseudoQC
- **R** (Implementation language for pseudoQC; environment in which the correction and normalization workflow is executed)

## Evaluation signals

- Per-method, per-dataset metrics are written to correction_normalization_comparison.csv with one row per regression method per dataset
- Nonlinear regression methods yield better correction/normalization metrics (lower residual signal drift and lower relative standard deviation of QC/internal-standard features) than linear methods, reproducing the paper's finding
- All four machine learning-based regression methods (two linear, two nonlinear) were run and produced simulated QC data for both the metabolomics and proteomics datasets
- Simulated QC data are generated for the same feature set as the published input intensity tables before correction is applied

## Limitations

- The article does not individually name the four machine learning-based regression methods used in pseudoQC (missing signal in the extracted methods section)
- Only two published datasets (one metabolomics, one proteomics) are demonstrated; performance on other datasets must be verified independently
- Operating details (parameters, expected inputs) are only in pseudoQC.Supplementary.pdf, not the main text

## Evidence

- [abstract] Nonlinear outperforms linear for correction/normalization: [evidence span withheld — anchor status: not_found]
- [abstract] Four ML-based regression methods in pseudoQC: "simulate QC sample data for actual metabolomics and proteomics datasets using four different machine learning-based regression methods"
- [methods] pseudoQC tool description and R implementation: "pseudoQC is a powerful and interactive web-based software to simulate QC sample data for actual metabolomics and proteomics datasets"
- [readme] Operating procedure location: [evidence span withheld — anchor status: not_found]
