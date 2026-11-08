---
name: plsda-class-discrimination-modeling
description: Use when a two-or-more-group metabolomics dataset (e.g. cancer vs. control
  m/z peak table with metadata) needs class-discrimination assessment, separation
  p-values, and feature loading rankings.
when_to_use_negative:
- The goal is unsupervised exploratory clustering without a class label — use PCA
  or t-SNE instead, which MetaboShiny offers in the same dimension-reduction panel.
- Input is raw, non-normalized peak data or a peak list with a high missing-value
  percentage above the user-set threshold — MetaboShiny requires normalization and
  imputation (e.g. Random Forest) before PLS-DA.
- The dataset has no metadata grouping variable to define classes (e.g. no cancer/control
  column), since PLS-DA is supervised and requires the group assignment.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MetaboShiny
  role: Provides the interactive PLS-DA dimension-reduction panel that runs the supervised
    class-discrimination model on normalized m/z peak tables and exports loadings
    and separation significance.
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: R
  role: Language in which MetaboShiny and its MetaboAnalystR-backed statistics are
    implemented; PLS-DA is executed through the R-based application.
  repo: https://github.com/joannawolthuis/MetaboShiny
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:dc5f2eeac35c6789a9d6742c4303b10d9d71d656604672ba9bb4f80dda6399f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannawolthuis/MetaboShiny
  tools:
  - MetaboShiny
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# plsda-class-discrimination-modeling

## Summary

Builds a partial least squares-discriminant analysis (PLS-DA) supervised dimension-reduction model to test whether pre-defined sample groups (e.g. lung cancer vs. control urine DI-MS samples) separate significantly, and ranks discriminatory m/z features via PLS-DA loadings.

## When to use

Use when a two-or-more-group metabolomics dataset (e.g. cancer vs. control m/z peak table with metadata) needs class-discrimination assessment, separation p-values, and feature loading rankings.

## When NOT to use

- The goal is unsupervised exploratory clustering without a class label — use PCA or t-SNE instead, which MetaboShiny offers in the same dimension-reduction panel.
- Input is raw, non-normalized peak data or a peak list with a high missing-value percentage above the user-set threshold — MetaboShiny requires normalization and imputation (e.g. Random Forest) before PLS-DA.
- The dataset has no metadata grouping variable to define classes (e.g. no cancer/control column), since PLS-DA is supervised and requires the group assignment.

## Inputs

- Normalized m/z peak table from MetaboShiny (post filtering, normalization, transformation, scaling, and missing-value imputation, e.g. Random Forest imputation)
- MetaboShiny-format metadata table with a grouping/experimental variable column (e.g. cancer vs. control status) and 'sample'/'individual' columns
- Merged MetaboShiny project loaded from a saved project name (e.g. the MTBLS28 lung cancer urine DI-MS dataset)

## Outputs

- PLS-DA separation significance (p-value) between defined sample groups
- PLS-DA variance explained and m/z loading rankings (PC1 loadings)
- Exported results table (e.g. plsda_separation_results.csv) containing the t-test table, PLS-DA variance/loadings, and separation p-value

## How to apply

In MetaboShiny, first normalize the m/z peak table (filter, normalize, transform, scale) and impute missing values with Random Forest — MetaboShiny notes other normalization methods run substantially faster but RF was used to mirror the original Mathé et al. analysis. Select PLS-DA from the dimension-reduction panel (alongside PCA and t-SNE), supplying the grouping metadata column (e.g. lung cancer vs. control status for the MTBLS28 1005-patient urine dataset). The PLS-DA run, backed by MetaboAnalystR, outputs separation significance between groups; the reported acceptance criterion for the reproduced Mathé et al. case study was a separation p-value satisfying p < 0.03. Inspect the PC1 loadings to confirm the known cancer-predictive m/z values (264, 308, 441, 561 — with 264 m/z identified as creatine riboside) rank highly among discriminatory features. Export the results (t-test table, PLS-DA variance/loadings, separation p-value) for downstream annotation, which MetaboShiny performs only on significant hits.

## Related tools

- **MetaboShiny** (Provides the interactive PLS-DA dimension-reduction panel that runs the supervised class-discrimination model on normalized m/z peak tables and exports loadings and separation significance.) — https://github.com/joannawolthuis/MetaboShiny
- **R** (Language in which MetaboShiny and its MetaboAnalystR-backed statistics are implemented; PLS-DA is executed through the R-based application.) — https://github.com/joannawolthuis/MetaboShiny

## Examples

```
docker run -p 8080:8080 -v ~/MetaboShiny/:/root/MetaboShiny/:cached --rm -it jcwolthuis/metaboshiny:latest Rscript -e "MetaboShiny::start_metshi(inBrowser=F)"
```

## Evaluation signals

- The reproduced PLS-DA separation p-value for cancer vs. control must satisfy p < 0.03, as reported in the original Mathé et al. paper.
- The four known cancer-predictive m/z values (264, 308, 441, 561) should appear among the top PC1 loadings, validating that the model captures the published discriminatory features.
- The exported results file (e.g. plsda_separation_results.csv) must contain the t-test table, PLS-DA variance/loadings, and separation p-value.
- Normalization QC plots should show the pre- and post-normalized peak distributions for a randomly selected set of m/z values and samples, confirming the input to PLS-DA was properly normalized before modeling.

## Limitations

- PLS-DA is supervised and only meaningful when a class/group metadata variable exists; for unlabeled exploration MetaboShiny directs users to PCA or t-SNE instead.
- Other normalization methods are expected to run substantially faster than the Random Forest missing-value imputation used in the demonstration.
- MetaboShiny performs downstream data analysis directly on the m/z ratios and metabolite annotation occurs only on significant hits, so loading-ranked m/z values without significance may remain unannotated.
- MetaboShiny does not automatically update databases; users must re-build databases manually, which affects any compound annotation step that follows PLS-DA loading inspection.

## Evidence

- [methods] PLS-DA runs via MetaboAnalystR; separation p-value must satisfy p < 0.03 for the reproduced MTBLS28 case study: [evidence span withheld — anchor status: not_found]
- [methods] Dimension reduction offers PCA, PLS-DA, t-SNE in MetaboShiny: "various dimension reduction methods such as PCA and PLS-DA, and volcano plots"
- [methods] Four cancer-predictive m/z values include creatine riboside at 264 m/z: [evidence span withheld — anchor status: not_found]
- [methods] Random Forest imputation is used for missing values before analysis: "Missing values were imputed using Random Forest, other normalization methods are expected to run substantially faster"
- [readme] MetaboShiny provides the interactive PLS-DA panel and its repository: "Partial least squares-discriminant analysis (PLS-DA)](#partial-least-squares-discriminant-analysis--pls-da-)"
- [intro] MetaboShiny runs statistics on m/z ratios, annotating only significant hits: "MetaboShiny performs downstream data analysis directly on the m/z ratios and metabolite annotation occurs only on e.g. significant hits that result from the analysis."
- [methods] PLS-DA is demonstrated on the MTBLS28 lung cancer urine dataset: "we leveraged the LC–MS dataset on urine samples from 1005 patients with and without lung cancer"
