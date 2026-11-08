---
name: metabolomics-quality-control-boxplot-visualization
description: Use when a MetaboDiff MultiAssayExperiment object (Metabolon®-style metabolite
  measurements) has been imputed and vsn-normalized and you must visually verify near-constant
  variance and comparable per-sample distributions before differential analysis.
when_to_use_negative:
- The MultiAssayExperiment object lacks the 'norm' and 'norm_imputed' slots because
  normalize_met has not yet been run — run normalization first, the boxplot comparison
  is meaningless without the vsn-normalized panel.
- The goal is differential testing (Student's T-Test, ANOVA with Benjamini-Hochberg
  correction) or metabolic correlation network analysis rather than QC of the normalization
  step.
- Input is a raw vendor metabolite measurement table that has not passed the knn_impute
  40% cutoff and outlier-cluster removal QC chain.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MetaboDiff
  role: Provides the quality_plot diagnostic and normalize_met function whose vsn
    output the boxplots validate
  repo: http://github.com/andreasmock/MetaboDiff/
- name: R
  role: Runtime environment for MetaboDiff (requires R version 4.0.2 or higher)
- name: MultiAssayExperiment
  role: Container object holding the raw/imputed/norm/norm_imputed assay slots that
    quality_plot visualizes
- name: Metabolon®
  role: Commercial metabolomics platform whose measurements (e.g. 61 prostate cancers
    vs 25 normal specimens) form the input data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:561e71d50c33dd533bfc47bde0d5752a5a8fa52175474c3a8764c1da7daebbf7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/andreasmock/MetaboDiff
  tools:
  - MetaboDiff
  - Metabolon®
  - MultiAssayExperiment
  - R
  - vsn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# metabolomics-quality-control-boxplot-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Renders per-sample boxplots of raw, imputed, normalized, and imputed+normalized metabolomic measurements with MetaboDiff's quality_plot to visually confirm that variance stabilizing normalization (vsn) has made the variance nearly constant across the measurement spectrum.

## When to use

Use when a MetaboDiff MultiAssayExperiment object (Metabolon®-style metabolite measurements) has been imputed and vsn-normalized and you must visually verify near-constant variance and comparable per-sample distributions before differential analysis.

## When NOT to use

- The MultiAssayExperiment object lacks the 'norm' and 'norm_imputed' slots because normalize_met has not yet been run — run normalization first, the boxplot comparison is meaningless without the vsn-normalized panel.
- The goal is differential testing (Student's T-Test, ANOVA with Benjamini-Hochberg correction) or metabolic correlation network analysis rather than QC of the normalization step.
- Input is a raw vendor metabolite measurement table that has not passed the knn_impute 40% cutoff and outlier-cluster removal QC chain.

## Inputs

- MultiAssayExperiment object from knn_impute (cutoff 0.4) after remove_cluster outlier-cluster removal
- group_factor column name defining sample groups (e.g. 'tumor_normal')
- label_colors vector mapping group levels to colors (e.g. c('darkseagreen','dodgerblue'))

## Outputs

- Quality control figure with four panels of per-sample boxplots (raw, imputed, normalized, imputed+normalized), e.g. quality_plot.png
- Visual confirmation that panel D (imputed+normalized) shows near-constant variance across the study set

## How to apply

First ensure the upstream QC chain is done: knn_impute with a 40% (0.4) missingness cutoff and removal of the outlier k-means cluster via remove_cluster (e.g. remove cluster 1 with k=2 in the MetaboDiff example data). Run MetaboDiff's normalize_met function, which applies variance stabilizing normalization (vsn) so the variance remains nearly constant over the measured spectrum, adding the 'norm' and 'norm_imputed' slots to the MultiAssayExperiment object. Then invoke quality_plot on the normalized object with group_factor 'tumor_normal' and label_colors ('darkseagreen','dodgerblue') to render boxplots of raw, imputed, normalized, and imputed+normalized measurements per sample. Inspect panel D: the criterion is that the distribution of imputed-and-normalized measurements is comparable across the study set, with near-constant variance and no outlier sample dominating the measurement spectrum. Save the rendered figure (e.g. as quality_plot.png) as the QC record.

## Related tools

- **MetaboDiff** (Provides the quality_plot diagnostic and normalize_met function whose vsn output the boxplots validate) — http://github.com/andreasmock/MetaboDiff/
- **R** (Runtime environment for MetaboDiff (requires R version 4.0.2 or higher))
- **MultiAssayExperiment** (Container object holding the raw/imputed/norm/norm_imputed assay slots that quality_plot visualizes)
- **Metabolon®** (Commercial metabolomics platform whose measurements (e.g. 61 prostate cancers vs 25 normal specimens) form the input data)

## Examples

```
quality_plot(metabo_multi_assay, group_factor = "tumor_normal", label_colors = c("darkseagreen", "dodgerblue"))
```

## Evaluation signals

- Panel D of the quality plot shows per-sample distributions of imputed+normalized measurements that are comparable across the study set
- Variance is near-constant across the measurement spectrum in the imputed-and-normalized panel (the stated purpose of vsn)
- No outlier sample dominates the measurement spectrum after remove_cluster cluster removal
- The MultiAssayExperiment object gains 'norm' and 'norm_imputed' slots after normalize_met
- Rendered figure colors sample groups via label_colors ('darkseagreen','dodgerblue') for 'tumor_normal' and is saved as quality_plot.png

## Limitations

- MetaboDiff requires R version 4.0.2 or higher
- The QC plot only validates normalization after the recommended knn_impute cutoff of 40% (0.4) and outlier-cluster removal; results from other preprocessing may not meet the 'comparable distribution' criterion
- The repository README contains no changelog ('No changelog found'), so version-specific behavior of quality_plot may be undocumented

## Evidence
