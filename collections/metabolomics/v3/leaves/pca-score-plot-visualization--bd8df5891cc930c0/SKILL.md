---
name: pca-score-plot-visualization
description: Use when a LipidomicsExperiment (e.g. imported from Metabolomics Workbench
  study ST001111 with measure="Area") needs unsupervised quality control, and samples
  with significantly large dispersion in the PCA scores plot must be identified as
  outliers before further analysis.
when_to_use_negative:
- The goal is supervised discrimination against a clinical outcome — use OPLS(DA)
  or univariate/volcano analysis instead of an unsupervised PCA score plot
- Input is a raw Skyline export or numerical matrix that has not yet been converted
  to a LipidomicsExperiment with sample annotations — import and annotate first
- Lipids rather than samples are the QC target (e.g. filtering by %CV) — box plots
  and lipid filtering are the appropriate lipidr tools
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: lipidr
  role: Computes PCA via mva(method="PCA") and renders the interactive score plot
    colored by sample annotation for outlier detection
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Runtime environment in which the LipidomicsExperiment is loaded, PCA is run,
    and the cleaned .rds dataset is saved
- name: Metabolomics Workbench API
  role: Source of the public lipidomics study (ST001111) whose peak areas feed the
    PCA score plot
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:53a95b766f6eb1d816dfc07113b6e65b8bae9d787f5cfe842852fe0c58eae9fd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaboReport
  - Metabolomics Workbench API
  - R
  - R/Bioconductor
  - ggplot2
  - lipidr
  - lmm2met
  - ropls
  - stats
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pca-score-plot-visualization

## Summary

Generates PCA score plots from a lipidr LipidomicsExperiment to reveal sample clustering and outliers in lipidomics peak-area data, using lipidr's multivariate analysis and interactive plotting.

## When to use

Use when a LipidomicsExperiment (e.g. imported from Metabolomics Workbench study ST001111 with measure="Area") needs unsupervised quality control, and samples with significantly large dispersion in the PCA scores plot must be identified as outliers before further analysis.

## When NOT to use

- The goal is supervised discrimination against a clinical outcome — use OPLS(DA) or univariate/volcano analysis instead of an unsupervised PCA score plot
- Input is a raw Skyline export or numerical matrix that has not yet been converted to a LipidomicsExperiment with sample annotations — import and annotate first
- Lipids rather than samples are the QC target (e.g. filtering by %CV) — box plots and lipid filtering are the appropriate lipidr tools

## Inputs

- LipidomicsExperiment object (e.g. imported, name-corrected Metabolomics Workbench study ST001111 lipid peak areas)
- Sample annotation table mapping sample names to groups (e.g. SampleType), with sample names matching the experiment

## Outputs

- Interactive PCA score plot colored by SampleType (saved as figure/*.png)
- List of outlier samples identified by large dispersion (e.g. samples 42 and 18 in ST001111)
- Cleaned LipidomicsExperiment with outliers removed (e.g. cleaned_st001111.rds)

## How to apply

Run lipidr::mva on the LipidomicsExperiment with measure="Area" and method="PCA" to compute principal components over lipid peak areas. Plot the PCA scores interactively with points colored by a sample annotation such as SampleType (loaded via add_sample_annotation). Inspect the score plot for samples with significantly large dispersion — in ST001111 breast-cancer tissue data this surfaced samples 42 and 18, which were then removed from the experiment. Rationale: lipidr generates PCA score plots specifically for quality control of samples and measured lipids, and outliers must be removed before downstream normalization, univariate, or enrichment analyses. Save the score plot (figure/*.png) and the cleaned dataset (cleaned_st001111.rds) so outlier removal is reproducible.

## Related tools

- **lipidr** (Computes PCA via mva(method="PCA") and renders the interactive score plot colored by sample annotation for outlier detection) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Runtime environment in which the LipidomicsExperiment is loaded, PCA is run, and the cleaned .rds dataset is saved)
- **Metabolomics Workbench API** (Source of the public lipidomics study (ST001111) whose peak areas feed the PCA score plot)

## Examples

```
pca <- mva(d, measure = "Area", method = "PCA"); plot(pca, color_by = "SampleType")
```

## Evaluation signals

- The score plot renders with points colored by SampleType and sample labels, allowing visual identification of dispersed points
- Outlier samples flagged visually (42 and 18 for ST001111) match samples with significantly large dispersion in the PCA scores
- After removing outliers, the cleaned experiment (cleaned_st001111.rds) no longer contains those sample names and the remaining clustering tightens
- The PCA was computed on measure="Area" values of the name-corrected experiment, not on normalized or filtered data
- The figure PNG is saved alongside the cleaned dataset so the outlier decision is traceable

## Limitations

- PCA on raw peak areas is sensitive to outliers and scaling choices; it is a QC/screening step, not evidence of class separation
- Outlier calls are visual judgments from the score plot; ambiguous dispersion should be cross-checked before removing samples
- PCoA and OPLS(DA) are also available in lipidr and may be better suited when an outcome variable of interest is known

## Evidence

- [readme] lipidr implements PCA, PCoA and OPLS(DA) to reveal patterns in data and discover variables related to an outcome of interest.: [evidence span withheld — anchor status: not_found]
- [readme] Top associated lipids as well as scores and loadings plots can be interactively investigated using `lipidr`.: [evidence span withheld — anchor status: external_doc]
