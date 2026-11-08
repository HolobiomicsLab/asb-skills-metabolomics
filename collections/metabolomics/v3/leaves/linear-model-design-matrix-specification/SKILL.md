---
name: linear-model-design-matrix-specification
description: Use when a lipidomics dataset has grouped samples plus a potential confounding
  clinical variable (e.g. Race, batch, age) and you need a covariate-adjusted group
  contrast rather than a plain two-group comparison.
when_to_use_negative:
- When there is no confounding variable and a simple two-group contrast suffices —
  the unadjusted comparison is appropriate and the design matrix adds a covariate
  term unnecessarily
- When the clinical variables of interest have not been added to the object via add_sample_annotation
  — de_design cannot build the design matrix from unloaded annotations
- When the analysis target is lipid set enrichment (LSEA) or multivariate ordination
  (PCA/OPLS-DA) rather than a linear-model contrast
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: lipidr
  role: Provides de_design to fit covariate-adjusted linear models on a LipidomicsExperiment
    and volcano plots of the coefficients
  repo: github.com/ahmohamed/lipidr
- name: limma
  role: Underlying linear-model/design-matrix machinery; lipidr points users to the
    Limma User Guide for more complex design matrices
- name: R/Bioconductor
  role: Runtime environment in which lipidr and the formula-based model fitting are
    executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:e310cdd2f6604a615dcdbbb633860a5d1cbe2d9e9153a0adfda4d8fe441734cd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R/Bioconductor
  - limma
  - lipidr
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# linear-model-design-matrix-specification

## Summary

Specify covariate-adjusted linear models for lipidomics differential analysis using lipidr's de_design, which builds design matrices over loaded clinical variables so a coefficient of interest (e.g. SampleTypeCancer) is estimated while controlling for confounders such as Race, with statistical power borrowed from the limma framework.

## When to use

Use when a lipidomics dataset has grouped samples plus a potential confounding clinical variable (e.g. Race, batch, age) and you need a covariate-adjusted group contrast rather than a plain two-group comparison.

## When NOT to use

- When there is no confounding variable and a simple two-group contrast suffices — the unadjusted comparison is appropriate and the design matrix adds a covariate term unnecessarily
- When the clinical variables of interest have not been added to the object via add_sample_annotation — de_design cannot build the design matrix from unloaded annotations
- When the analysis target is lipid set enrichment (LSEA) or multivariate ordination (PCA/OPLS-DA) rather than a linear-model contrast

## Inputs

- LipidomicsExperiment (lipidr object) built from a numerical matrix (lipids as rows, samples as columns) or a Skyline CSV export, with sample annotation CSV containing clinical variables (e.g. SampleType, Race)
- R formula defining the design (e.g. ~ Race + SampleType) and a coefficient name for the contrast of interest

## Outputs

- Differential-analysis table per molecule with log2 fold change and adjusted p-values for the chosen coefficient (exportable as CSV)
- Volcano plot (log2 fold change vs statistical significance) for the covariate-adjusted contrast

## How to apply

Load the dataset as a LipidomicsExperiment (e.g. from a numerical matrix or Skyline CSV export with add_sample_annotation for the clinical table), then fit the model with lipidr's de_design using an R formula that includes both the confounder and the outcome, e.g. de_design(d, ~ Race + SampleType, coef="SampleTypeCancer"), so the Cancer-vs-Benign contrast is estimated adjusted for Race. Choose the coefficient string to match the contrast encoded in the design matrix (e.g. SampleTypeCancer). Extract log2 fold changes and adjusted p-values from the limma-based results, filter significant molecules at the standard adjusted-p threshold, and export the table plus a volcano plot of log2FC versus significance. For more complex designs (interactions, multi-group), lipidr defers to the limma User Guide for custom design matrices. Validate by checking that the adjusted contrast reproduces the pattern of the unadjusted two-group comparison — in the ST001111 example, a similar pattern implied only a small Race effect.

## Related tools

- **lipidr** (Provides de_design to fit covariate-adjusted linear models on a LipidomicsExperiment and volcano plots of the coefficients) — github.com/ahmohamed/lipidr
- **limma** (Underlying linear-model/design-matrix machinery; lipidr points users to the Limma User Guide for more complex design matrices)
- **R/Bioconductor** (Runtime environment in which lipidr and the formula-based model fitting are executed)

## Examples

```
de_design(d, ~ Race + SampleType, coef="SampleTypeCancer")
```

## Evaluation signals

- Design matrix is built from clinical variables present in the sample annotation (SampleType, Race) and the coef argument resolves to a valid column (e.g. SampleTypeCancer)
- Adjusted results show a molecular pattern similar to the two-group SampleType-only comparison when the confounder effect is small (expert-review validation from task_007)
- Result table contains log2FC and adjusted p-values per lipid molecule, filtered to molecules below the standard adjusted-p significance threshold before CSV export
- Volcano plot axes correspond to log2 fold change and statistical significance of the specified coefficient
- Formula terms match loaded clinical variables exactly; an unmatched variable name in ~ Race + SampleType should raise an error rather than silently drop

## Limitations

- Formula-based designs are limited by the clinical variables loaded into the sample annotation; complex custom designs require consulting the Limma or edgeR user guides
- Interpretation of a 'small confounder effect' is comparative — it relies on the adjusted contrast reproducing the unadjusted two-group pattern, which requires an existing baseline comparison
- Significance filtering depends on the standard adjusted-p threshold; rare/confounded factor levels (e.g. sparse Race categories) can destabilize coefficient estimates

## Evidence

- [readme] Multi-group comparisons and adjusting for confounding variables is also supported: [evidence span withheld — anchor status: external_doc]
