---
name: differential-lipid-abundance-analysis
description: Use when a targeted lipidomics dataset (Skyline CSV export or numerical
  matrix with sample-annotation table) must be statistically compared across clinical
  groups, e.g. HighFat_water vs NormalDiet_water or Cancer vs Benign, with or without
  confounder correction.
when_to_use_negative:
- Data has not yet been imported, transition-summarized, or normalized — run lipidr
  QC/normalization (box plots, PCA, %CV filtering, PQN) first, since differential
  results depend on those steps
- The goal is lipid set enrichment (preferential regulation of lipid classes, total
  chain lengths, or unsaturation patterns) rather than per-molecule abundance testing
  — use lipidr's LSEA instead
- There is no clinical grouping variable or contrast of interest loaded in the sample
  annotation — de_analysis/de_design require loaded clinical variables to build the
  model
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: lipidr
  role: R/Bioconductor package providing read_skyline/add_sample_annotation import,
    de_analysis and de_design (limma-based differential testing on loaded clinical
    variables), and volcano-plot visualization
  repo: github.com/ahmohamed/lipidr
- name: limma
  role: Underlying linear-model framework used by lipidr's de_analysis/de_design to
    fit contrasts and covariate-adjusted design matrices; users needing more complex
    designs are referred to the Limma User Guide
- name: Skyline
  role: Source of the exported CSV transition-level results that read_skyline imports
    as the analysis input
merged_aliases:
- roi-wise-differential-lipid-abundance-analysis
merged_alias_records:
- alias: lipid-differential-abundance-analysis
  slug: lipid-differential-abundance-analysis
  jaccard_score: 1.0
  method: token-set-jaccard
  decision: auto
- alias: lipid-differential-expression-analysis
  slug: lipid-differential-expression-analysis
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_008
  - task_007
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:d4cc07d96f30d0d6f6982fbecf46e0be736f65a721ab39c638a88dcaf4006317
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Skyline
  - limma
  - lipidr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# differential-lipid-abundance-analysis

## Summary

Statistical comparison of lipid-molecule abundance between clinical groups using lipidr's limma-based de_analysis/de_design on a LipidomicsExperiment, with results exported as logFC/p-value tables and volcano plots. This is the core re-analysis capability that lipidr provides for targeted lipidomics datasets, including covariate-adjusted (factorial) designs.

## When to use

Use when a targeted lipidomics dataset (Skyline CSV export or numerical matrix with sample-annotation table) must be statistically compared across clinical groups, e.g. HighFat_water vs NormalDiet_water or Cancer vs Benign, with or without confounder correction.

## When NOT to use

- Data has not yet been imported, transition-summarized, or normalized — run lipidr QC/normalization (box plots, PCA, %CV filtering, PQN) first, since differential results depend on those steps
- The goal is lipid set enrichment (preferential regulation of lipid classes, total chain lengths, or unsaturation patterns) rather than per-molecule abundance testing — use lipidr's LSEA instead
- There is no clinical grouping variable or contrast of interest loaded in the sample annotation — de_analysis/de_design require loaded clinical variables to build the model

## Inputs

- Skyline-exported results CSV (e.g. A1_data.csv from lipidr inst/extdata)
- Numerical matrix CSV (lipids as rows, samples as columns, e.g. brca_matrix.csv)
- Sample annotation / clinical-variables CSV with sample names in the first column (e.g. clin.csv, brca_clin.csv)
- LipidomicsExperiment object (transition- or molecule-level) with sample annotations attached

## Outputs

- Differential-analysis results table (logFC and p-values per lipid molecule) exported as CSV, one per contrast or coefficient
- Volcano plots of log2 fold change versus statistical significance per contrast, saved as PNG
- Filtered list of significant lipid molecules (adjusted p-value below the significance threshold)

## How to apply

Import the data into a LipidomicsExperiment via read_skyline() (Skyline-exported CSV) or as_lipidomics_experiment(read.csv(...)) (numerical matrix), then attach the sample annotation with add_sample_annotation(d, "clin.csv") so clinical variables are available for modeling. Summarize transition-level measurements to the lipid-molecule level and normalize (e.g. PQN) before testing so sample-to-sample intensity differences do not drive fold changes. For two-group contrasts call de_analysis with the desired contrast (e.g. HighFat_water - NormalDiet_water, measure="Area"); for confounder-adjusted or factorial designs use de_design with a formula such as ~ Race + SampleType and coef="SampleTypeCancer", which fits limma linear models on the loaded clinical variables. Export per-lipid logFC and p-values as CSV, filter to significant molecules by adjusted p-value threshold, and visualize each contrast with lipidr's volcano plot. Correct application is judged by whether the exported tables and volcano plots reproduce expected regulation patterns — e.g. in ST001111, PCs and PGs up-regulated and CLs and TGs down-regulated in cancer vs benign, and covariate-adjusted results showing a pattern similar to the two-group comparison when confounder effects are small.

## Related tools

- **lipidr** (R/Bioconductor package providing read_skyline/add_sample_annotation import, de_analysis and de_design (limma-based differential testing on loaded clinical variables), and volcano-plot visualization) — github.com/ahmohamed/lipidr
- **limma** (Underlying linear-model framework used by lipidr's de_analysis/de_design to fit contrasts and covariate-adjusted design matrices; users needing more complex designs are referred to the Limma User Guide)
- **Skyline** (Source of the exported CSV transition-level results that read_skyline imports as the analysis input)

## Examples

```
d <- read_skyline("Skyline_export.csv"); d <- add_sample_annotation(d, "clin.csv"); de_res <- de_analysis(d, HighFat_water - NormalDiet_water, measure="Area")
```

## Evaluation signals

- Exported DE table contains per-lipid logFC and p-values for every specified contrast (e.g. Cancer-Benign and Cancer-Metasis; HighFat_water - NormalDiet_water with measure="Area")
- Volcano plot per contrast renders log2 fold change vs statistical significance; when the number of significantly altered lipids is large, labeling is omitted (as in the diet-dataset vignette)
- Class-level regulation pattern check: in ST001111 cancer vs benign, PCs and PGs up-regulated and CLs and TGs down-regulated; only minute differences between cancer and metastasis
- Covariate-adjusted factorial test (de_design ~ Race + SampleType, coef="SampleTypeCancer") shows a pattern similar to the two-group SampleType-only comparison, indicating a small Race effect
- Significant-molecule subset extracted by adjusted p-value below the standard significance threshold before CSV export

## Limitations

- When a large number of lipids are significantly altered (e.g. HighFat_water vs NormalDiet_water), volcano-plot labeling becomes impractical and is omitted
- Multi-group comparisons and confounder adjustment are supported but require constructing design matrices; users interested in more complex designs are referred to the Limma User Guide (and edgeR tutorial)
- Sample names must be identical in the data matrix and the sample-annotation table, or add_sample_annotation will fail

## Evidence

- [intro] Differential analysis can be performed using any of the loaded clinical variables, which can be readily visualized as volcano plots.: "Differential analysis can be performed using any of the loaded clinical variables, which can be readily visualized as volcano plots."
- [readme] Multi-group comparisons and adjusting for confounding variables is also supported (refer to examples on www.lipidr.org).: [evidence span withheld — anchor status: external_doc]
- [readme] Univariate analysis can be performed using any of the loaded clinical variables, which can be readily visualized as volcano plots.: "Univariate analysis can be performed using any of the loaded clinical variables, which can be readily visualized as volcano plots."
- [intro] Users interested in creating more complex design matrices are referred to [Limma User Guide]: "Users interested in creating more complex design matrices are referred to [Limma User Guide]"
- [intro] `lipidr` takes exported Skyline CSV as input, allowing for multiple methods to be analyzed together.: "`lipidr` takes exported Skyline CSV as input, allowing for multiple methods to be analyzed together."
