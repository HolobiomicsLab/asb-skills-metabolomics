---
name: proteomics-qc-sample-simulation
description: Use when a metabolomics or proteomics feature/sample intensity matrix
  lacks measured pooled-QC injections and drift correction/normalization still requires
  QC-reference signal profiles.
when_to_use_negative:
- The dataset already contains real measured pooled-QC sample injections, which are
  preferable to simulated QC data
- The input is not a feature/sample intensity matrix (e.g. raw instrument files or
  sequence data), since pseudoQC simulates QC data from actual metabolomics and proteomics
  datasets
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: pseudoQC
  role: Interactive web-based software that simulates QC sample data for actual metabolomics
    and proteomics datasets using four machine learning-based regression methods
  repo: https://www.omicsolution.org/wukong/pseudoQC/
- name: R
  role: Language in which pseudoQC is developed; the R environment is used to run
    the software and downstream correction/normalization of the datasets
  repo: https://github.com/qade544/pseudoQC
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201900264
    title: 'pseudoQC: A Regression‐Based Simulation Software for Correction and Normalization
      of Complex Metabolomics and Proteomics Datasets'
  source_content_hash: sha256:f625cda04ea2691ac6dce83344b291c76309b6f3d4eef0d004aaa70fc0067f46
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - pseudoQC
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# proteomics-qc-sample-simulation

## Summary

Regression-based simulation of pooled-QC sample data for real metabolomics or proteomics feature-intensity datasets using pseudoQC, an open-source R/web software implementing four machine learning-based regression methods; the simulated QC data then drive signal correction and normalization.

## When to use

Use when a metabolomics or proteomics feature/sample intensity matrix lacks measured pooled-QC injections and drift correction/normalization still requires QC-reference signal profiles.

## When NOT to use

- The dataset already contains real measured pooled-QC sample injections, which are preferable to simulated QC data
- The input is not a feature/sample intensity matrix (e.g. raw instrument files or sequence data), since pseudoQC simulates QC data from actual metabolomics and proteomics datasets

## Inputs

- Feature/sample intensity matrix from a published or new metabolomics dataset
- Feature/sample intensity matrix from a published or new proteomics dataset
- Operating procedure document pseudoQC.Supplementary.pdf from the GitHub repository

## Outputs

- One simulated pooled-QC sample data table per regression method (four tables total), retaining the original feature/sample identifiers
- Corrected and normalized feature intensity tables produced using each simulated QC dataset
- Per-method, per-dataset correction/normalization performance metrics (e.g. residual signal drift, relative standard deviation of QC/internal-standard features)

## How to apply

Load the feature/sample intensity matrix of the metabolomics or proteomics dataset into pseudoQC (R-based, web interface at omicsolution.org/wukong/pseudoQC/). For each of the four machine learning-based regression methods implemented in the software, run the regression-based simulation to generate a simulated pooled-QC sample table from the actual dataset, preserving original feature and sample identifiers so one simulated QC table is exported per method (four tables total). Follow the detailed operating procedure in pseudoQC.Supplementary.pdf. Then apply each set of simulated QC samples to perform correction and normalization of the corresponding real dataset; the paper's reported result is that nonlinear regression methods outperform linear ones, so the per-method simulation is a decision point for method selection.

## Related tools

- **pseudoQC** (Interactive web-based software that simulates QC sample data for actual metabolomics and proteomics datasets using four machine learning-based regression methods) — https://www.omicsolution.org/wukong/pseudoQC/
- **R** (Language in which pseudoQC is developed; the R environment is used to run the software and downstream correction/normalization of the datasets) — https://github.com/qade544/pseudoQC

## Evaluation signals

- Exactly four simulated QC data tables are produced, one per machine learning-based regression method (two linear, two nonlinear)
- Each simulated QC table contains the same features as the input dataset, so it can be applied to correction and normalization of that dataset
- The software reports a successful run via the web interface at https://www.omicsolution.org/wukong/pseudoQC/
- Per-method performance metrics (e.g. residual signal drift, QC relative standard deviation) reproduce the paper's result that nonlinear regression methods outperform linear ones
- Simulated QC data can be applied to both published datasets (metabolomics and proteomics) without identifier mismatch

## Limitations

- The four machine learning-based regression methods are not individually named in the article's methods section, so method identities must be confirmed from the software or supplementary PDF
- The article does not report quantitative thresholds or cutoffs for choosing between linear and nonlinear methods; comparison relies on computed performance metrics
- pseudoQC is a web-based tool, so batch/programmatic invocation details are limited to the supplementary operating procedure

## Evidence

- [abstract] a powerful and interactive web-based software, named pseudoQC, is presented to simulate QC sample data for actual metabolomics and proteomics datasets using four different machine learning-based: "a powerful and interactive web-based software, named pseudoQC, is presented to simulate QC sample data for actual metabolomics and proteomics datasets using four different machine learning-based"
- [abstract] the simulated data are used for correction and normalization of the two published datasets, and the obtained results suggest that nonlinear regression methods perform better than linear ones: [evidence span withheld — anchor status: not_found]
- [readme] It is developed with R and the software can also be found here: https://www.omicsolution.org/wukong/pseudoQC/: [evidence span withheld — anchor status: not_found]
