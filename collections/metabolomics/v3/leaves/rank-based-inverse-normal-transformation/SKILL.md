---
name: rank-based-inverse-normal-transformation
description: Use when a metabolite abundance table (e.g., SMART narcotics or breast
  cancer demonstrative data) shows high variance and non-normally distributed metabolite
  abundances that would compromise downstream association or classification analysis.
when_to_use_negative:
- The abundance table is already variance-stabilized and approximately normally distributed,
  e.g., after an equivalent prior normalization step in the SMART preprocessing module
- The goal is targeted quantification accuracy (concentration calibration) or peak
  identification rather than statistical normalization of abundance distributions
- Quality control, batch effect analysis, or peak detection issues (e.g., low S/N
  peaks like amphetamine with S/N ratio 0.19) are the actual problem — normalization
  will not fix signal detection defects
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: SMART 2.0
  role: Provides the RINT normalization module (with complementary Pareto scaling)
    in its data preprocessing module; written in R
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: Predecessor tool whose data preprocessing workflow SMART 2.0 extends with
    the new normalization methods
  repo: https://github.com/YuJenL/SMART
- name: R
  role: Runtime environment in which the SMART package and its RINT transformation
    are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:c611695ba2b33b1e0f0a87e1875e9c0c384b8891faef7a9cc63aadb3ad252e47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - R
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# rank-based-inverse-normal-transformation

## Summary

Apply RINT, a new data normalization module in SMART 2.0, to metabolite abundance tables so each metabolite's distribution is mapped to a standard normal scale, stabilizing variance and facilitating downstream statistical analysis alongside Pareto scaling (PS).

## When to use

Use when a metabolite abundance table (e.g., SMART narcotics or breast cancer demonstrative data) shows high variance and non-normally distributed metabolite abundances that would compromise downstream association or classification analysis.

## When NOT to use

- The abundance table is already variance-stabilized and approximately normally distributed, e.g., after an equivalent prior normalization step in the SMART preprocessing module
- The goal is targeted quantification accuracy (concentration calibration) or peak identification rather than statistical normalization of abundance distributions
- Quality control, batch effect analysis, or peak detection issues (e.g., low S/N peaks like amphetamine with S/N ratio 0.19) are the actual problem — normalization will not fix signal detection defects

## Inputs

- metabolite abundance table (e.g., SMART example narcotics or breast cancer demonstrative data)
- unprocessed/normalized-lacking feature table from targeted or untargeted metabolomics analysis

## Outputs

- PS- and RINT-processed abundance table written to ps_rint_transformed_table.csv
- pre- vs post-transformation comparison metrics (variance homogeneity, distribution shape)

## How to apply

Load the metabolite abundance table shipped with the SMART package (narcotics or breast cancer demonstrative data) into R and run SMART 2.0's data preprocessing module, which introduces PS and RINT as complementary methods. First apply the PS transformation to reduce the influence of large fold changes while preserving the data structure, then apply the RINT normalization module to rank-transform each metabolite's abundance distribution onto a standard normal scale. Numerically compare the pre- and post-transformation tables — checking variance homogeneity across samples and distribution shape — to confirm that preprocessing achieved variance stabilization and normalization before proceeding to association analysis (ANCOVA), PLS/PLS-DA classification, or IOPA.

## Related tools

- **SMART 2.0** (Provides the RINT normalization module (with complementary Pareto scaling) in its data preprocessing module; written in R) — https://github.com/YuJenL/SMART
- **SMART 1.0** (Predecessor tool whose data preprocessing workflow SMART 2.0 extends with the new normalization methods) — https://github.com/YuJenL/SMART
- **R** (Runtime environment in which the SMART package and its RINT transformation are executed)

## Evaluation signals

- ps_rint_transformed_table.csv exists with the expected sample/metabolite structure matching the input table
- Numerical comparison shows improved variance homogeneity across samples relative to the pre-transformation table
- Post-RINT metabolite abundance distributions approximate a standard normal shape per metabolite
- Transformed values remain rank-consistent with the original abundance ordering (RINT is rank-based, so monotone ordering within each metabolite is preserved)

## Limitations

- No changelog was found for SMART 2.0, so version-to-version behavior of the preprocessing module is undocumented
- RINT addresses distribution shape and variance; it does not resolve underlying data-quality problems such as low S/N peaks or inconsistent LC-MS experimental conditions
- Any remaining structure-quality concerns after transformation are referred to expert review rather than automatically resolved

## Evidence

- [abstract] abstract: "introducing new analytical modules in targeted metabolomics analysis, data normalization, quality control assessment, and advanced dimensionality reduction and classification methods"
- [other] other: [evidence span withheld — anchor status: not_found]
- [readme] readme: "Data Preprocessing: Perform data transformation and standardization."
- [discussion] discussion: "The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART."
