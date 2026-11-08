---
name: volcano-plot-visualization
description: 'Use when a batch-corrected metabolomics feature table (samples in rows,
  features in columns) needs a differential-abundance visual summary: fit a per-feature
  linear model, BH-adjust p-values, and render a volcano plot with Log2FC > 1.5 and
  adj-p < 0.05 cutoffs.'
when_to_use_negative:
- Input data is raw, uncorrected LC-MS intensities — batch effect correction (e.g.
  via dbnorm's ber) must be applied first, as raw data analysis can miss metabolites
  significantly altered by HFD.
- The goal is to identify which batch-correction model best fits the data — dbnormSCORE
  / Visodbnorm, not the volcano plot, is the appropriate diagnostic step.
- The analysis is a batch-effect QC (e.g. PCA, RLA, hclustdbnorm) rather than differential
  abundance testing.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: dbnorm
  role: Provides the batch-effect-corrected feature table on which the volcano plot's
    differential analysis is performed; the article applies dbnorm's statistical models
    (ber, ber-bagging, parametric/non-parametric ComBat) before the Log2FC/adj-p test.
  repo: https://github.com/NBDZ/dbnorm
- name: ggplot2
  role: Used in R to render the volcano plot graphic (log2FC vs -log10 adj-p) and
    export it to PNG.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:bbcf0f90101738451eb5f3d3c543d95b6e51f867c92cb7d906ba967822bd1e50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - GetFeatistics
  - R
  - dbnorm
  - ggplot2
  - omu
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# volcano-plot-visualization

## Summary

Generates a volcano plot from the dbnorm-corrected mouse adipose LC-MS metabolomics dataset, plotting log2 fold change against -log10(adjusted p-value) to visualize metabolites significantly altered by high-fat diet using the thresholds Log2FC > 1.5 and adj-p < 0.05.

## When to use

Use when a batch-corrected metabolomics feature table (samples in rows, features in columns) needs a differential-abundance visual summary: fit a per-feature linear model, BH-adjust p-values, and render a volcano plot with Log2FC > 1.5 and adj-p < 0.05 cutoffs.

## When NOT to use

- Input data is raw, uncorrected LC-MS intensities — batch effect correction (e.g. via dbnorm's ber) must be applied first, as raw data analysis can miss metabolites significantly altered by HFD.
- The goal is to identify which batch-correction model best fits the data — dbnormSCORE / Visodbnorm, not the volcano plot, is the appropriate diagnostic step.
- The analysis is a batch-effect QC (e.g. PCA, RLA, hclustdbnorm) rather than differential abundance testing.

## Inputs

- CSV feature table of batch-corrected metabolomics intensities (independent experiments in rows, features/variables in columns, batch levels in the first column)
- Sample metadata mapping samples to diet condition (HFD vs control)

## Outputs

- Volcano plot PNG of log2 fold change vs -log10(adjusted p-value)
- CSV list of significant metabolites (Log2FC > 1.5, BH-adjusted p < 0.05)

## How to apply

Load the feature table (e.g. the ber-corrected sc-AT 8-week HFD vs control data exported as .csv from dbnorm), fit a linear regression model per metabolic feature to estimate fold change and p-value of the diet effect, and apply Benjamini-Hochberg correction for multiple testing. Select significant features with Log2FC > 1.5 and BH-adjusted p < 0.05, then draw a volcano plot of Log2FC against -log10(adjusted p-value) for all features, highlighting the significant subset. Export the significant feature list to CSV and the plot to PNG. The rationale, grounded in the article, is that batch correction (ber) yields a more exhaustive candidate list than raw data, so the volcano plot must be built on corrected intensities to avoid confounding technical drift with biological diet effect.

## Related tools

- **dbnorm** (Provides the batch-effect-corrected feature table on which the volcano plot's differential analysis is performed; the article applies dbnorm's statistical models (ber, ber-bagging, parametric/non-parametric ComBat) before the Log2FC/adj-p test.) — https://github.com/NBDZ/dbnorm
- **ggplot2** (Used in R to render the volcano plot graphic (log2FC vs -log10 adj-p) and export it to PNG.)

## Examples

```
Rscript volcano_analysis.R --input ber_corrected_scAT.csv --log2fc 1.5 --padj 0.05 --out volcano.png
```

## Evaluation signals

- Number of significant features (Log2FC > 1.5 and BH-adjusted p < 0.05) should be non-zero; the article reports a more exhaustive candidate list after ber correction than raw data.
- Volcano plot x-axis must be log2 fold change (or log2FC) and y-axis -log10(adjusted p-value); points beyond the cutoffs are the highlighted significant subset.
- Exported CSV should contain only features meeting both thresholds; any feature with adjusted p >= 0.05 or |Log2FC| <= 1.5 must be absent.
- Adjusted p-values must come from Benjamini-Hochberg correction, not raw per-feature p-values.
- The plot must be based on batch-corrected intensities (e.g. ber output), not raw uncorrected data.

## Limitations

- The specific Log2FC > 1.5 and adj-p < 0.05 thresholds and volcano-plot workflow are documented in the article's HFD/adipose case study; they are not generic defaults — other experiments may need different cutoffs.
- dbnorm functions like Visodbnorm and dbnormSCORE are recommended for fewer than 2000 features for computational speed; very large untargeted feature tables (e.g. the >20,000 m/z values from XCMS) may need the decomposed functions (dbnormBer, etc.).
- The dbnorm R package requires log2-scaled input data; raw intensities must be log2-transformed and scaled before correction, and the volcano analysis inherits this requirement.
- Residual batch association can persist with some correction models (lowess, nonparametric ComBat), which may distort fold-change and p-value estimates feeding the volcano plot.

## Evidence

- [results] Volcano plot differential analysis thresholds: "Significant criteria of Log2FC > 1.5 and adj-p-value < 0.05 are considered to draw a list of metabolites with differential level."
- [results] batch correction before differential analysis: [evidence span withheld — anchor status: not_found]
- [abstract] residual association in some models: [evidence span withheld — anchor status: not_found]
- [readme] log2 scaling input requirement: "Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features (variables)"
- [readme] feature count limits for dbnorm functions: "This function is suggested for less than 2000 features (variables) for better computational speed."
