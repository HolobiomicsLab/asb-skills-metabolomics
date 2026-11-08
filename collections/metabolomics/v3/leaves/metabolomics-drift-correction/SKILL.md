---
name: metabolomics-drift-correction
description: Use when a metabolomics or proteomics feature-intensity table lacks real
  QC samples and signal drift correction/normalization must be driven by simulated
  QC data, or when benchmarking linear vs nonlinear regression for correction.
when_to_use_negative:
- The dataset already contains real, measured QC samples — apply QC-based signal correction
  directly rather than simulating QC data.
- Input is raw instrument data (e.g., mzML or vendor RAW files) rather than a processed
  feature-intensity table; pseudoQC operates on intensity tables.
- The goal is feature detection or identification — this skill only covers signal
  correction and normalization of an existing intensity matrix.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2269
tools:
- name: pseudoQC
  role: Interactive web-based software (developed in R) that simulates QC sample data
    with four machine learning-based regression methods and uses them for signal correction
    and normalization of the intensity table
  repo: https://github.com/qade544/pseudoQC
- name: R
  role: Language/environment in which pseudoQC is implemented; the skill's simulation
    and correction workflow runs in the R-based pseudoQC environment
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201900264
    title: 'pseudoQC: A Regression‐Based Simulation Software for Correction and Normalization
      of Complex Metabolomics and Proteomics Datasets'
  source_content_hash: sha256:2be4c10cd2e3affc4ed2a22fddd00c873bd3a206032bd5bb73128248400f6ca7
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolomics-drift-correction

## Summary

Simulates pseudo quality-control (QC) sample data for metabolomics and proteomics feature-intensity tables using four machine learning-based regression methods, then applies the simulated QC for signal drift correction and normalization. It addresses the common case where real QC samples are unavailable, and the paper's benchmark shows nonlinear regressors outperform linear ones for this correction.

## When to use

Use when a metabolomics or proteomics feature-intensity table lacks real QC samples and signal drift correction/normalization must be driven by simulated QC data, or when benchmarking linear vs nonlinear regression for correction.

## When NOT to use

- The dataset already contains real, measured QC samples — apply QC-based signal correction directly rather than simulating QC data.
- Input is raw instrument data (e.g., mzML or vendor RAW files) rather than a processed feature-intensity table; pseudoQC operates on intensity tables.
- The goal is feature detection or identification — this skill only covers signal correction and normalization of an existing intensity matrix.

## Inputs

- Feature-intensity table from a published metabolomics dataset (features x samples)
- Feature-intensity table from a published proteomics dataset (features x samples)
- pseudoQC.Supplementary.pdf operating procedure

## Outputs

- Simulated (pseudo) QC sample data, one set per regression method (four per dataset)
- Corrected and normalized feature-intensity tables per regression method
- Per-method, per-dataset performance metrics (residual signal drift, RSD of QC/internal-standard features) written to correction_normalization_comparison.csv

## How to apply

Load the published metabolomics or proteomics feature-intensity table into the R-based pseudoQC environment, following the operating procedure in pseudoQC.Supplementary.pdf (web interface at https://www.omicsolution.org/wukong/pseudoQC/). Simulate QC sample data for the dataset with the four machine learning-based regression methods implemented in pseudoQC (two linear and two nonlinear regressors). Use each set of simulated QC samples to perform signal correction and normalization of the corresponding real dataset. For every method compute performance metrics such as residual signal drift and the relative standard deviation (RSD) of QC/internal-standard features, writing per-method, per-dataset values (e.g., to correction_normalization_comparison.csv). Judge success by comparing linear vs nonlinear metrics across the datasets; the paper's reported result is that nonlinear regression methods perform better than linear ones, so a linear method winning on these data signals a likely execution problem.

## Related tools

- **pseudoQC** (Interactive web-based software (developed in R) that simulates QC sample data with four machine learning-based regression methods and uses them for signal correction and normalization of the intensity table) — https://github.com/qade544/pseudoQC
- **R** (Language/environment in which pseudoQC is implemented; the skill's simulation and correction workflow runs in the R-based pseudoQC environment) — https://www.r-project.org/

## Evaluation signals

- Nonlinear regressors yield lower residual signal drift and lower RSD of QC/internal-standard features than the two linear regressors on the same dataset.
- Corrected/normalized outputs preserve the input feature-by-sample dimensions (no features lost or added).
- correction_normalization_comparison.csv contains one metric row per regression method per dataset, covering all four methods.
- Each run first generates simulated QC sample data for all four machine learning-based regression methods before correction/normalization.
- Reproducing the paper's two published datasets recovers the reported conclusion that nonlinear regression methods perform better than linear ones.

## Limitations

- The four regression methods are not individually named in the article; they are described only generically as machine learning-based regressors (two linear and two nonlinear per the source workflow).
- The detailed operating procedure exists only in pseudoQC.Supplementary.pdf; no command-line interface is documented, so usage is via the web interface or R.
- Correction quality rests on simulated rather than measured QC samples, and the evaluation covered only two published datasets (one metabolomics, one proteomics).

## Evidence

- [abstract] Nonlinear regression outperforms linear for correction and normalization of the two published datasets: [evidence span withheld — anchor status: not_found]
- [methods] QC simulation uses four machine learning-based regression methods: "simulate QC sample data for actual metabolomics and proteomics datasets using four different machine learning–based regression methods"
- [methods] Detailed usage documented only in the supplementary PDF: "can be found in pseudoQC.Supplementary.pdf (https://github.com/qade544/pseudoQC/blob/master/pseudoQC.Supplementary.pdf) file"
- [readme] pseudoQC is implemented in R and available as a web app: [evidence span withheld — anchor status: not_found]
