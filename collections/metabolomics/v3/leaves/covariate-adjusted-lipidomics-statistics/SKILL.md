---
name: covariate-adjusted-lipidomics-statistics
description: Use when a lipidomics dataset (e.g. Metabolomics Workbench study ST001111)
  has loaded clinical variables and the differential-abundance contrast may be confounded
  by a covariate such as Race, requiring a factorial model like ~ Race + SampleType.
when_to_use_negative:
- The analysis is a simple two-group comparison with no confounding variable to adjust
  for — use lipidr's univariate two-group differential analysis instead of a factorial
  de_design model.
- The data are not lipidomics intensities with sample annotation (e.g. already a feature
  table without clinical variables loaded) — the design matrix must be built from
  loaded clinical variables.
- The dataset lacks any loaded clinical covariates; de_design requires sample annotation
  containing both the coefficient of interest and the adjustment covariate.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: lipidr
  role: R/Bioconductor package providing de_design for covariate-adjusted differential
    analysis of LipidomicsExperiment objects and volcano-plot visualization of results
  repo: https://github.com/ahmohamed/lipidr
- name: limma
  role: Underlying linear-model framework; users creating more complex design matrices
    are referred to the Limma User Guide
- name: R/Bioconductor
  role: Execution environment in which the lipidr LipidomicsExperiment is loaded,
    modeled, and exported to CSV/PNG
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:6e60dda935e89c0ccf99472cc6020b3a638df79d5c4152e57781040e9ea2aaee
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ahmohamed/lipidr
  tools:
  - R/Bioconductor
  - limma
  - lipidr
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# covariate-adjusted-lipidomics-statistics

## Summary

Perform covariate-adjusted differential abundance analysis of lipid molecules in a lipidomics dataset using lipidr's limma-based de_design function, so the effect of a primary variable (e.g. SampleType Cancer vs Benign) is estimated while correcting for confounders (e.g. Race), then export significant molecules as CSV and inspect them with a volcano plot.

## When to use

Use when a lipidomics dataset (e.g. Metabolomics Workbench study ST001111) has loaded clinical variables and the differential-abundance contrast may be confounded by a covariate such as Race, requiring a factorial model like ~ Race + SampleType.

## When NOT to use

- The analysis is a simple two-group comparison with no confounding variable to adjust for — use lipidr's univariate two-group differential analysis instead of a factorial de_design model.
- The data are not lipidomics intensities with sample annotation (e.g. already a feature table without clinical variables loaded) — the design matrix must be built from loaded clinical variables.
- The dataset lacks any loaded clinical covariates; de_design requires sample annotation containing both the coefficient of interest and the adjustment covariate.

## Inputs

- Cleaned lipidomics intensity matrix (lipids as rows, samples as columns) as CSV or Skyline-exported CSV
- Sample annotation/clinical table (CSV) with sample names matching the matrix, containing the outcome variable (e.g. SampleType) and the confounding covariate (e.g. Race)
- Metabolomics Workbench study file (e.g. ST001111) importable via lipidr

## Outputs

- CSV table of differential-analysis results filtered to significant molecules (adjusted p-value below the significance threshold)
- Volcano plot PNG of log2 fold change versus statistical significance for the adjusted coefficient (e.g. SampleTypeCancer)
- Comparison/interpretation vs the unadjusted two-group comparison quantifying the size of the covariate effect

## How to apply

Load the cleaned lipidomics dataset into R as a lipidr LipidomicsExperiment (e.g. via as_lipidomics_experiment + add_sample_annotation, or read_skyline for Skyline exports), ensuring sample names match between the intensity matrix and the clinical annotation file. Fit the covariate-adjusted linear model with lipidr's de_design(d, ~ Race + SampleType, coef="SampleTypeCancer"), which builds a limma-style design matrix so the SampleTypeCancer contrast is estimated while adjusting for the Race covariate; lipidr supports multi-group comparisons and adjusting for confounding variables using any loaded clinical variable. Extract the differential-analysis results, filter to significant molecules (adjusted p-value below the standard significance threshold, e.g. FDR < 0.05), and export the table as a CSV. Generate a volcano plot of log2 fold change versus statistical significance for the SampleTypeCancer coefficient with lipidr's volcano-plot function and save it as a PNG. Validate by expert review that the covariate-adjusted results show a pattern similar to the two-group (SampleType-only) comparison, indicating only a small Race effect.

## Related tools

- **lipidr** (R/Bioconductor package providing de_design for covariate-adjusted differential analysis of LipidomicsExperiment objects and volcano-plot visualization of results) — https://github.com/ahmohamed/lipidr
- **limma** (Underlying linear-model framework; users creating more complex design matrices are referred to the Limma User Guide)
- **R/Bioconductor** (Execution environment in which the lipidr LipidomicsExperiment is loaded, modeled, and exported to CSV/PNG)

## Examples

```
results <- lipidr::de_design(d, ~ Race + SampleType, coef = "SampleTypeCancer")
```

## Evaluation signals

- The covariate-adjusted (SampleType-only) results show a pattern similar to the two-group comparison, indicating a small Race effect — expert review confirms consistency of significant lipid directionality.
- Significant molecules are filtered by adjusted p-value below the standard significance threshold before export to CSV.
- The volcano plot renders log2 fold change versus statistical significance for the SampleTypeCancer coefficient and is saved as PNG.
- Sample names are identical between the intensity matrix and the clinical annotation file, so the design matrix builds without errors.
- The estimated log2FC for SampleTypeCancer comes from a design including the Race term, i.e. ~ Race + SampleType, not from a SampleType-only model.

## Limitations

- Adjustment depends on the covariate being present as a loaded clinical variable in the sample annotation; missing or unmatched sample names break the design.
- No changelog was found for the repository, so behavioral changes across lipidr versions may not be traceable.
- Interpretation of 'small confounder effect' relies on expert review comparing adjusted vs unadjusted patterns rather than a formal quantitative test.

## Evidence

- [intro] Differential analysis can be performed using any of the loaded clinical variables, which can be readily visualized as volcano plots.: "Differential analysis can be performed using any of the loaded clinical variables, which can be readily visualized as volcano plots."
- [readme] Multi-group comparisons and adjusting for confounding variables is also supported: [evidence span withheld — anchor status: external_doc]
- [readme] a table with the sample annotation / groups, where the sample names are in first column. Note the sample names must be identical in the two files.: [evidence span withheld — anchor status: external_doc]
- [other] Fit the covariate-adjusted linear model with lipidr's de_design(d, ~ Race + SampleType, coef="SampleTypeCancer"): [evidence span withheld — anchor status: not_found]
- [intro] The example shown in this vignette is from `ST001111`: "The example shown in this vignette is from `ST001111`"
