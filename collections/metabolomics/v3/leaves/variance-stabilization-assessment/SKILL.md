---
name: variance-stabilization-assessment
description: Use when a metabolite abundance table exhibits high between-sample variance
  or non-normally distributed metabolite abundances that would distort downstream
  association or classification statistics.
when_to_use_negative:
- Input is not a metabolite feature/abundance table (e.g., raw instrument files such
  as .raw, .d, or mzXML that first need peak analysis in SMART's Peak Analysis module)
- Abundances are already variance-stabilized and approximately normal, so the transformed
  table would offer no numerical improvement
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: SMART 2.0
  role: R package whose Data Preprocessing module implements PS and RINT transformation/standardization
    of metabolite abundance tables
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: predecessor tool providing the baseline preprocessing-to-association-analysis
    workflow that SMART 2.0's normalization modules extend
  repo: https://github.com/YuJenL/SMART
- name: R
  role: runtime environment in which the SMART package is executed for the PS/RINT
    transformations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:aa34d0096ea114ccc966fc4e24c5c9072eafa904b98976fdedb4999f2f11d840
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - R
  - SMART 1.0
  - SMART 2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# variance-stabilization-assessment

## Summary

Apply SMART 2.0's Pareto scaling (PS) and rank-based inverse normal transformation (RINT) to metabolite abundance tables, then numerically compare pre- and post-transformation data to verify variance stabilization and approximate normality.

## When to use

Use when a metabolite abundance table exhibits high between-sample variance or non-normally distributed metabolite abundances that would distort downstream association or classification statistics.

## When NOT to use

- Input is not a metabolite feature/abundance table (e.g., raw instrument files such as .raw, .d, or mzXML that first need peak analysis in SMART's Peak Analysis module)
- Abundances are already variance-stabilized and approximately normal, so the transformed table would offer no numerical improvement

## Inputs

- metabolite abundance table (e.g., SMART narcotics or breast cancer demonstrative data)
- R environment with the SMART 2.0 package installed

## Outputs

- PS- and RINT-processed abundance table (ps_rint_transformed_table.csv)
- numerical pre/post comparison of variance homogeneity and distribution shape

## How to apply

Load the metabolite abundance table (e.g., the narcotics or breast cancer demonstrative data shipped with SMART) into R and apply SMART 2.0's Data Preprocessing module, which performs data transformation and standardization. First apply Pareto scaling (PS) to the metabolite abundance columns to reduce the influence of large fold changes while preserving the data structure. Then apply the rank-based inverse normal transformation (RINT) to map each metabolite's abundance distribution onto a standard normal scale, stabilizing variance and approximating normality. Numerically compare pre- and post-transformation tables (variance homogeneity across samples, distribution shape) to confirm the transformations achieved their goals, then write the result to ps_rint_transformed_table.csv; refer any remaining structure-quality concerns to expert review.

## Related tools

- **SMART 2.0** (R package whose Data Preprocessing module implements PS and RINT transformation/standardization of metabolite abundance tables) — https://github.com/YuJenL/SMART
- **SMART 1.0** (predecessor tool providing the baseline preprocessing-to-association-analysis workflow that SMART 2.0's normalization modules extend) — https://github.com/YuJenL/SMART
- **R** (runtime environment in which the SMART package is executed for the PS/RINT transformations)

## Evaluation signals

- ps_rint_transformed_table.csv exists with the expected sample/metabolite structure matching the input table
- pre/post numerical comparison shows improved variance homogeneity across samples after PS
- post-RINT metabolite abundance distributions approximate a standard normal scale
- any residual structure-quality concerns are explicitly flagged for expert review rather than silently accepted

## Limitations

- SMART 2.0 has no changelog, so version differences in preprocessing behavior are not documented
- Acceptance of the transformed table is heuristic (numerical comparison of variance/distribution shape); remaining structure-quality concerns require expert review
- Quality control for samples and peaks is a separate SMART module; transformation alone does not address poor-quality peaks (e.g., low S/N detections like amphetamine's S/N ratio of 0.19)

## Evidence

- [abstract] PS reduces the influence of large fold changes while preserving the data structure, and RINT helps stabilize variance and approximate normality to facilitate downstream statistical analysis.: "PS reduces the influence of large fold changes while preserving the data structure, and RINT helps stabilize variance and approximate normality"
- [intro] we significantly enhance the capacities of SMART 1.0 by introducing advanced analytical modules: "we significantly enhance the capacities of SMART 1.0 by introducing advanced analytical modules"
- [readme] Data Preprocessing: Perform data transformation and standardization.: "Data Preprocessing: Perform data transformation and standardization."
- [readme] Quality Control: Conduct quality control for samples and peaks.: "Quality Control: Conduct quality control for samples and peaks."
