---
name: metabolomics-workbench-dataset-reanalysis
description: Use when the goal is to retrieve and reanalyze a public Metabolomics
  Workbench lipidomics study (e.g. ST001111) in R with lipidr, performing differential
  or multivariate analysis on public lipid intensity data.
when_to_use_negative:
- Input is already a fully analyzed results table (logFC/p-values) rather than raw
  lipid intensity data requiring import and modeling
- The dataset is not a lipidomics study and lipid names cannot be parsed for class/chain
  information
- Only a single group of samples is available, so two-group or factorial contrasts
  cannot be fitted
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: lipidr
  role: Core R/Bioconductor package providing dataset import, LipidomicsExperiment
    construction, normalization, limma-based differential analysis, LSEA, and OPLS-DA
  repo: github.com/ahmohamed/lipidr
- name: Metabolomics Workbench API
  role: Source for searching, downloading, and reanalyzing public lipidomics datasets
    such as ST001111
- name: limma
  role: Linear-model engine underlying de_analysis/de_design for two-group and covariate-adjusted
    contrasts
- name: Skyline
  role: 'Alternative input format: lipidr reads Skyline-exported CSV files alongside
    a sample annotation CSV'
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:5b1486b67de148c6d1bbb98c4e5ac8f94bc32a24c02faf7d4740204fd294db8b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Metabolomics Workbench API
  - Skyline
  - limma
  - lipidr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# metabolomics-workbench-dataset-reanalysis

## Summary

A lipidr-based workflow for searching, downloading, and reanalyzing public lipidomics datasets through the Metabolomics Workbench API inside R/Bioconductor, producing LipidomicsExperiment objects that support normalization, differential analysis (limma-based), and OPLS-DA.

## When to use

Use when the goal is to retrieve and reanalyze a public Metabolomics Workbench lipidomics study (e.g. ST001111) in R with lipidr, performing differential or multivariate analysis on public lipid intensity data.

## When NOT to use

- Input is already a fully analyzed results table (logFC/p-values) rather than raw lipid intensity data requiring import and modeling
- The dataset is not a lipidomics study and lipid names cannot be parsed for class/chain information
- Only a single group of samples is available, so two-group or factorial contrasts cannot be fitted

## Inputs

- Metabolomics Workbench study ID (e.g. ST001111) retrieved via the Metabolomics Workbench API
- Numerical lipid intensity matrix (lipids as rows, samples as columns, lipid names in first column) or Skyline CSV export
- Sample annotation / clinical variable table (sample names in first column, matching the matrix column names)
- Model formula and contrast specification for de_analysis / de_design (e.g. coef="SampleTypeCancer")

## Outputs

- LipidomicsExperiment object (extends SummarizedExperiment) with lipid class and chain annotations inferred from lipid names
- Differential analysis results table (logFC and p-values per lipid) exported as CSV
- Volcano plots of log2 fold change versus statistical significance per contrast
- OPLS-DA loadings ranking with annotated top lipids (e.g. top10_oplsda_lipids.csv, exactly 10 rows with lipid name and class annotation)

## How to apply

Retrieve the study of interest through the Metabolomics Workbench API (e.g. study ST001111) and import the numerical matrix plus sample annotation table with as_lipidomics_experiment and add_sample_annotation, ensuring sample names are identical in both files; this builds a LipidomicsExperiment extending SummarizedExperiment. Perform quality control (PCA/box plots, filtering lipids by %CV) and optionally normalize with or without internal standards. Run limma-based differential analysis with de_analysis for two-group contrasts (e.g. Cancer-Benign, Cancer-Metastasis) or de_design for covariate-adjusted factorial models (e.g. ~ Race + SampleType), then export logFC and p-values as CSV and visualize as volcano plots. For multivariate exploration, use mva(method="OPLS-DA", group_col="SampleType", groups=c("Benign","Cancer")) and rank top contributing lipids with plot_mva_loadings and top_lipids. Validate by checking that class-level regulation patterns (e.g. PCs and PGs up-regulated, CLs and TGs down-regulated in cancer vs benign) are reproduced in the exported tables and plots.

## Related tools

- **lipidr** (Core R/Bioconductor package providing dataset import, LipidomicsExperiment construction, normalization, limma-based differential analysis, LSEA, and OPLS-DA) — github.com/ahmohamed/lipidr
- **Metabolomics Workbench API** (Source for searching, downloading, and reanalyzing public lipidomics datasets such as ST001111)
- **limma** (Linear-model engine underlying de_analysis/de_design for two-group and covariate-adjusted contrasts)
- **Skyline** (Alternative input format: lipidr reads Skyline-exported CSV files alongside a sample annotation CSV)

## Examples

```
d <- as_lipidomics_experiment(read.csv("data_matrix.csv")); d <- add_sample_annotation(d, "data_clin.csv"); de <- de_design(d, ~ Race + SampleType, coef="SampleTypeCancer")
```

## Evaluation signals

- LipidomicsExperiment is constructed successfully with sample annotation added and sample names identical in the matrix and clinical files
- Exported differential-analysis CSV contains logFC and p-values per lipid and reproduces the reported class regulation pattern (PCs/PGs up, CLs/TGs down in cancer vs benign; minute differences cancer vs metastasis)
- Covariate-adjusted de_design (~ Race + SampleType) results show a pattern similar to the two-group comparison, indicating a small Race effect
- OPLS-DA top-lipid table contains exactly 10 rows, each with a lipid name and class annotation, ordered by loading ranking
- Volcano plots are generated per contrast and match the limma-fitted coefficients

## Limitations

- Sample names must be identical between the data matrix and the annotation file or add_sample_annotation will fail
- lipidr infers lipid class and chain information from lipid names, so non-standard lipid name conventions may break annotation
- Confounding variables (e.g. Race) must be present as loaded clinical variables to be included in de_design models

## Evidence

- [readme] README: multivariate analysis scope: [evidence span withheld — anchor status: external_doc]
