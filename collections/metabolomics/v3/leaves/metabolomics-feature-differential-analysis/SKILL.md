---
name: metabolomics-feature-differential-analysis
description: Use when a UPLC-HRMS metabolomics feature table (samples × m/z features)
  must be tested per feature for a two-group diet/treatment effect and the candidate
  list of significantly altered metabolites (mostly lipids) must be reported with
  a volcano plot.
when_to_use_negative:
- When the input is already annotated or aggregated to the pathway/metabolite-name
  level and the goal is enrichment analysis (MSEA) rather than per-feature differential
  testing.
- When batch/technical drift has not been assessed and the signal is dominated by
  batch effects — the article's PCA and dbnormSCORE workflow should first be applied
  to choose a correction model; differential testing on raw data may miss diet-affected
  metabolites.
- When the design is a multi-factor or longitudinal study rather than a simple two-group
  contrast (e.g. HFD vs control), since the skill's linear regression and Log2FC >
  1.5 cutoff are tuned for a single diet effect.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: dbnorm
  role: Provides pre-processing, missing value estimation (emvd/emvf) and batch effect
    correction (dbnormBer, dbnormBagging, dbnormPcom, dbnormNPcom) of the feature
    matrix prior to differential analysis; dbnormSCORE picks the best-fitting model
    by adjusted-R² score.
  repo: https://github.com/NBDZ/dbnorm
- name: R
  role: Environment in which the differential regression per feature, BH adjustment
    and volcano plot generation are implemented.
- name: ggplot2
  role: Used to draw the volcano plot of Log2FC versus -log10(adjusted p-value) with
    the significant features highlighted.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:2cc635038bc578b11c3d8f25e5fa97103a6e95ddd0372807ced53b50c9e6ab18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - R
  - dbnorm
  - ggplot2
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# metabolomics-feature-differential-analysis

## Summary

Per-feature differential analysis of LC-MS metabolomics data between two conditions (e.g. 8-week high-fat diet vs. control mouse adipose tissue), selecting altered metabolic features via a Log2FC > 1.5 and BH-adjusted p-value < 0.05 cutoff and visualizing them on a volcano plot, as performed on batch-corrected (dbnorm) or raw data.

## When to use

Use when a UPLC-HRMS metabolomics feature table (samples × m/z features) must be tested per feature for a two-group diet/treatment effect and the candidate list of significantly altered metabolites (mostly lipids) must be reported with a volcano plot.

## When NOT to use

- When the input is already annotated or aggregated to the pathway/metabolite-name level and the goal is enrichment analysis (MSEA) rather than per-feature differential testing.
- When batch/technical drift has not been assessed and the signal is dominated by batch effects — the article's PCA and dbnormSCORE workflow should first be applied to choose a correction model; differential testing on raw data may miss diet-affected metabolites.
- When the design is a multi-factor or longitudinal study rather than a simple two-group contrast (e.g. HFD vs control), since the skill's linear regression and Log2FC > 1.5 cutoff are tuned for a single diet effect.

## Inputs

- Feature-level LC-MS metabolomics table in .csv format with independent experiments (samples) in rows and metabolic features (m/z values) in columns
- Batch level column (e.g. first column) identifying analytical batch/run order per sample
- Sample grouping metadata specifying the diet contrast (e.g. 8-week HFD vs control) for each sample

## Outputs

- List of significantly altered metabolites (CSV) with Log2FC, raw p-value and BH-adjusted p-value per feature
- Volcano plot (PNG/PDF) plotting Log2FC against -log10(adjusted p-value) with the significant subset highlighted
- Optional: pathway-level enrichment of significant metabolites via MSEA against KEGG pathways

## How to apply

Load the feature-level metabolomics data matrix as .csv with samples in rows and features (m/z variables) in columns, optionally after batch effect correction with the dbnorm statistical models (ber, ber-bagging, parametric or nonparametric ComBat) selected by the dbnormSCORE adjusted-R² score. Fit a linear regression model per metabolic feature to estimate the Log2 fold change of the diet (HFD vs control) effect and its p-value, then apply Benjamini-Hochberg (BH) adjustment across all features for multiple testing. Select significant features using the article's explicit criteria — Log2FC > 1.5 and adjusted p-value < 0.05 — to draw the list of metabolites with differential level. Produce a volcano plot (Log2FC against -log10(adjusted p-value)) with the significant subset highlighted, and optionally annotate the significant metabolites to functional pathways via metabolite set enrichment analysis (MSEA) mapped against KEGG pathways. Comparing raw vs. corrected analyses is the article's core use: after ber correction, several metabolites (predominantly lipids) became significant that were missed in the raw data, so run both and diff the candidate lists.

## Related tools

- **dbnorm** (Provides pre-processing, missing value estimation (emvd/emvf) and batch effect correction (dbnormBer, dbnormBagging, dbnormPcom, dbnormNPcom) of the feature matrix prior to differential analysis; dbnormSCORE picks the best-fitting model by adjusted-R² score.) — https://github.com/NBDZ/dbnorm
- **R** (Environment in which the differential regression per feature, BH adjustment and volcano plot generation are implemented.)
- **ggplot2** (Used to draw the volcano plot of Log2FC versus -log10(adjusted p-value) with the significant features highlighted.)

## Examples

```
data <- read.csv("path/to/mydata.csv", sep = ",", header = T, row.names = 1); library(dbnorm); dbnormSCORE(data)
```

## Evaluation signals

- The reported candidate list matches the dual cutoff: every exported feature has Log2FC > 1.5 AND BH-adjusted p-value < 0.05 — no feature outside these bounds appears in the significant list.
- The volcano plot's highlighted subset is exactly the significant list, with the axes reading Log2FC (x) and -log10(adjusted p-value) (y).
- Adjusted p-values are monotonically consistent with raw p-values under BH correction and all lie in [0, 1]; raw p-values are computed from the same per-feature linear regression model.
- When the same test is run on raw vs. dbnorm ber-corrected data, the corrected dataset yields an equal or larger significant candidate list; features newly significant only after correction should be inspectable and mostly identifiable as lipids.
- The output CSV contains one row per tested metabolic feature with the same number of columns (feature, Log2FC, p-value, adjusted p-value) across raw and corrected runs, enabling a direct diff of the candidate lists.

## Limitations

- The article does not specify a dbnorm software version used; version-to-version differences in ber/ComBat implementations may change the corrected matrix and thus the differential result.
- The dbnorm package's batch-effect-adjustment and visualization functions (Visodbnorm, dbnormSCORE) are suggested for less than 2000 features (variables) for better computational speed, so very large untargeted feature tables may need the decomposed functions (dbnormBer etc.).
- Lowess and nonparametric ComBat corrections can leave residual association of metabolite intensities to batch level (up to 78% and 60% residual variability respectively in the SKIPOGH dataset), so using these models before differential testing may still leak batch into the diet effect estimate.
- The Log2FC > 1.5 / adj-p < 0.05 thresholds are the article's criteria for the HFD mouse adipose tissue contrast; they are not a universal significance standard and should be justified per study.

## Evidence

- [results] Significant criteria of Log2FC > 1.5 and adj-p-value < 0.05 are considered to draw a list of metabolites with differential level.: "Significant criteria of Log2FC > 1.5 and adj-p-value < 0.05 are considered to draw a list of metabolites with differential level."
- [abstract] dbnorm assigns a score (adj-R2, 0 to 1) to help users identify the best fitting model for each dataset.: [evidence span withheld — anchor status: not_found]
- [abstract] ber corrected data showed no association of metabolite intensities to batch level, while some residual association remained in lowess- and nonparametric ComBat-corrected data.: [evidence span withheld — anchor status: not_found]
- [readme] Data to be uploaded must be normalized and scaled on the log2; input data must be in .csv format with experiments in rows and features in columns, with batch levels considered in the first column.: "The input data must be in **.csv** format with the independent experiments in the rows and the features (variables) in the columns, with the `batch` levels considered in the first column."
- [readme] Visodbnorm and dbnormSCORE functions are suggested for less than 2000 features (variables) for better computational speed.: "This function is suggested for less than 2000 features (variables) for better computational speed."
- [results] PCA plot of raw data generated by dbnorm showed that the samples were clustered by batch order.: "The Principal Component Analysis (PCA) plot of raw data generated by “dbnorm” showed that the samples were clustered by batch order (Fig. 1A)."
