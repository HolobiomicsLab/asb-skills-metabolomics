---
name: metabolomics-data-preprocessing
description: Use when a metabolite abundance table shows high variance between samples
  or non-normally distributed metabolite abundances, and you need variance stabilization
  and normalization before association or classification analysis in SMART 2.0.
when_to_use_negative:
- Input is raw instrument data (.raw, .d, mzXML) requiring peak analysis or annotation
  rather than transformation of an existing abundance table — use SMART's Data Import
  / Peak Analysis modules first
- The abundance table is already variance-stabilized and approximately normally distributed,
  so additional PS/RINT transformation would not improve downstream analysis
- The goal is batch effect analysis or post-analysis tasks (peak identification, concentration
  calibration), which are separate SMART 2.0 modules
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SMART 2.0
  role: Provides the Data Preprocessing module with PS (Pareto scaling) and RINT (rank-based
    inverse normal transformation) for metabolite abundance tables
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: Foundational R tool whose integrated untargeted metabolomics workflow (data
    preprocessing through association analysis) SMART 2.0 extends
  repo: https://github.com/YuJenL/SMART
- name: R
  role: Runtime environment in which SMART is executed (SMART written in R and R GUI)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:c8ac9b94e2a4b7ab9a6e4ac98b1ad7d0b5f9f14e32dd8a3c015d4049fa405bbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - R
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# metabolomics-data-preprocessing

## Summary

Applies SMART 2.0's data preprocessing module to metabolite abundance tables using Pareto scaling (PS) and rank-based inverse normal transformation (RINT), stabilizing between-sample variance and approximating normality for downstream statistical analysis.

## When to use

Use when a metabolite abundance table shows high variance between samples or non-normally distributed metabolite abundances, and you need variance stabilization and normalization before association or classification analysis in SMART 2.0.

## When NOT to use

- Input is raw instrument data (.raw, .d, mzXML) requiring peak analysis or annotation rather than transformation of an existing abundance table — use SMART's Data Import / Peak Analysis modules first
- The abundance table is already variance-stabilized and approximately normally distributed, so additional PS/RINT transformation would not improve downstream analysis
- The goal is batch effect analysis or post-analysis tasks (peak identification, concentration calibration), which are separate SMART 2.0 modules

## Inputs

- Metabolite abundance table (e.g., the narcotics or breast cancer demonstrative data shipped with the SMART package)
- R environment with the SMART 2.0 package installed

## Outputs

- PS- and RINT-processed abundance table (ps_rint_transformed_table.csv)
- Pre- vs. post-transformation numerical comparison (variance homogeneity and distribution shape)

## How to apply

Load the example metabolite abundance table shipped with the SMART package (narcotics or breast cancer demonstrative data) into R from the YuJenL/SMART repository. Apply SMART 2.0's Pareto scaling (PS) module to the metabolite abundance columns to reduce the influence of large fold changes while preserving the data structure. Then apply the RINT module, which performs a rank-based inverse normal transformation to map each metabolite's abundance distribution onto a standard normal scale. Numerically compare pre- and post-transformation tables — checking variance homogeneity across samples and distribution shape — to confirm that preprocessing achieved variance stabilization and normalization. This preprocessing step sits in SMART's Data Preprocessing module (module 4, 'data transformation and standardization'), which feeds downstream modules such as association analysis (ANCOVA) and classification (PLS/PLS-DA).

## Related tools

- **SMART 2.0** (Provides the Data Preprocessing module with PS (Pareto scaling) and RINT (rank-based inverse normal transformation) for metabolite abundance tables) — https://github.com/YuJenL/SMART
- **SMART 1.0** (Foundational R tool whose integrated untargeted metabolomics workflow (data preprocessing through association analysis) SMART 2.0 extends) — https://github.com/YuJenL/SMART
- **R** (Runtime environment in which SMART is executed (SMART written in R and R GUI))

## Examples

```
Rscript -e "library(SMART); dat <- read.csv('metabolite_abundance.csv'); out <- SMART::preprocess(dat, method=c('PS','RINT')); write.csv(out, 'ps_rint_transformed_table.csv')"
```

## Evaluation signals

- ps_rint_transformed_table.csv exists with the expected sample/metabolite structure matching the input table
- Numerical comparison shows improved variance homogeneity across samples after PS
- Post-RINT metabolite abundance distributions approximate a standard normal scale
- Transformed table retains the same dimensions (samples × metabolites) as the input abundance table
- Any remaining structure-quality concerns are referred to expert review rather than silently accepted

## Limitations

- No changelog found for SMART 2.0 (per the discussion section), so version-to-version preprocessing behavior differences are not documented
- Preprocessing does not fix upstream peak detection issues; e.g., low-abundance amphetamine peaks in the narcotics data may be mis-detected (S/N ratio of amphetamine was only 0.19) and LC-MS conditions need re-evaluation
- Consistency of LC-MS experimental conditions is required for reliable downstream calibration but is not guaranteed

## Evidence

- [full_text] PS reduces the influence of large fold changes while preserving the data structure, and RINT helps stabilize variance and approximate normality to facilitate downstream statistical analysis.: [evidence span withheld — anchor status: not_found]
- [readme] Data Preprocessing: Perform data transformation and standardization.: "4.	Data Preprocessing: Perform data transformation and standardization."
- [full_text] SMART 2.0's data preprocessing module introduces PS and RINT as complementary methods for metabolomics data with high variance and non-normally distributed metabolite abundances.: [evidence span withheld — anchor status: not_found]
- [abstract] introducing new analytical modules in targeted metabolomics analysis, data normalization, quality control assessment, and advanced dimensionality reduction and classification methods: "introducing new analytical modules in targeted metabolomics analysis, data normalization, quality control assessment, and advanced dimensionality reduction and classification methods"
