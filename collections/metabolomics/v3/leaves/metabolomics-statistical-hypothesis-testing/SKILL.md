---
name: metabolomics-statistical-hypothesis-testing
description: Use when you have the grouped molecular feature table from OUKS step
  8 (Grouping.R) plus a metadata.csv defining two sample groups, and you need hypothesis
  testing, fold changes, CV-based biomarker selection, and ROC/UMAP/t-SNE outputs.
when_to_use_negative:
- Input is raw LC-MS data (.CDF) rather than the grouped molecular feature table —
  upstream OUKS steps (1. Randomization.R through 8. Grouping.R, e.g. Integration,
  Imputation, Filtering) must run first
- The goal is intermediate preprocessing such as missing value imputation, signal
  drift correction, batch effect removal, or D-Ratio filtering — these belong to OUKS
  steps 3, 4, and 6
- The dataset has no two-group design (e.g. no bladder cancer vs control class labels
  in metadata.csv), since the univariate testing and ROC analysis assume group comparisons
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3577
tools:
- name: OUKS
  role: Provides the 9. Statistics.R script implementing the statistical analysis
    and hypothesis testing step, plus Figures for OUKS.R for ROC and ordination plotting
  repo: https://github.com/plyush1993/OUKS
- name: R (>= 4.1.2)
  role: Runtime required to execute the Statistics script and p.adjust() correction
  repo: https://cloud.r-project.org/
- name: RStudio
  role: IDE in which the tidy-format, headings-organized OUKS scripts are run
  repo: https://www.rstudio.com/products/rstudio/download/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00392
    title: '<i>Omics Untargeted Key Script</i>: R-Based Software Toolbox for Untargeted
      Metabolomics with Bladder Cancer Biomarkers Discovery Case Study'
  source_content_hash: sha256:5cf225c6ef90a79a48827adec1d7530f6e52ac2dc3dc687ee0ce1568d92e488f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/plyush1993/OUKS
  tools:
  - OUKS
  - R
  - R (>= 4.1.2)
  - RStudio
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# metabolomics-statistical-hypothesis-testing

## Summary

Final (step 9) analytical stage of the OUKS nine-step LC-MS untargeted metabolomics toolbox: per-feature univariate hypothesis testing with p.adjust()-based multiple-testing correction, fold-change computation, consensus/nested cross-validation biomarker selection, and ROC plus UMAP/t-SNE visualization for biomarker discovery (bladder cancer case study).

## When to use

Use when you have the grouped molecular feature table from OUKS step 8 (Grouping.R) plus a metadata.csv defining two sample groups, and you need hypothesis testing, fold changes, CV-based biomarker selection, and ROC/UMAP/t-SNE outputs.

## When NOT to use

- Input is raw LC-MS data (.CDF) rather than the grouped molecular feature table — upstream OUKS steps (1. Randomization.R through 8. Grouping.R, e.g. Integration, Imputation, Filtering) must run first
- The goal is intermediate preprocessing such as missing value imputation, signal drift correction, batch effect removal, or D-Ratio filtering — these belong to OUKS steps 3, 4, and 6
- The dataset has no two-group design (e.g. no bladder cancer vs control class labels in metadata.csv), since the univariate testing and ROC analysis assume group comparisons

## Inputs

- Grouped molecular feature table (.csv) output by OUKS 8. Grouping.R (peaks grouping and molecular features clustering)
- metadata.csv table (sample metadata defining bladder cancer vs control groups)
- R >= 4.1.2 environment (RStudio)

## Outputs

- Fold Change table (.csv) with per-feature fold change values
- Multiple-testing-corrected hypothesis testing results (p.adjust()-based adjusted p-values per molecular feature)
- Consensus/nested cross-validation feature-selection results ranking potential bladder cancer biomarkers
- ROC curve figures for selected potential biomarkers (with AUC) via Figures for OUKS.R
- UMAP/t-SNE ordination plots visualizing sample separation between groups

## How to apply

Load the grouped molecular feature table produced by 8. Grouping.R and the metadata.csv table into R (>= 4.1.2) via RStudio, then run the 9. Statistics.R script from the plyush1993/OUKS Scripts (R) folder. For every molecular feature, run univariate hypothesis testing between the bladder cancer and control groups and apply R's p.adjust() for multiple-testing correction (note: p.adjust() bugs in this script were only fixed in later versions). Compute group fold changes per feature and export the Fold Change table to .csv. Rank candidate biomarkers using consensus and nested cross-validation feature selection, then generate ROC curve figures for the selected potential biomarkers with the Figures for OUKS.R plotting code, together with UMAP/t-SNE ordination plots showing sample separation.

## Related tools

- **OUKS** (Provides the 9. Statistics.R script implementing the statistical analysis and hypothesis testing step, plus Figures for OUKS.R for ROC and ordination plotting) — https://github.com/plyush1993/OUKS
- **R (>= 4.1.2)** (Runtime required to execute the Statistics script and p.adjust() correction) — https://cloud.r-project.org/
- **RStudio** (IDE in which the tidy-format, headings-organized OUKS scripts are run) — https://www.rstudio.com/products/rstudio/download/

## Examples

```
Rscript "Scripts (R)/9. Statistics.R"
```

## Evaluation signals

- Each stated output exists and is well-formed: ROC figure, Fold Change .csv, UMAP/t-SNE ordination plot, and CV feature-selection results
- Fold Change .csv contains the expected metric fields: fold change and adjusted p-value per molecular feature
- ROC figures report AUC for the selected potential biomarkers
- Adjusted p-values are consistent with p.adjust()-based multiple-testing correction across all features
- Ordination plots show visible separation between bladder cancer and control groups consistent with the selected biomarker panel

## Limitations

- Earlier versions of OUKS contained bugs in the p.adjust() implementation affecting results in the 9. Statistics (and 7. Normalization/4. Correction) scripts; use a fixed release
- The project is marked Inactive (stable but no longer actively developed), with maintenance only as time allows
- Some required packages have strong version dependency; archives are supplied in the 'Required packages (archive)' folder, which can hinder reproducibility
- Windows OS is only desirable (not required), and the user is assumed to know basic R syntax

## Evidence

- [methods] Statistics (statistical analysis and hypothesis testing): "`9. Statistics.R` | statistical analysis and hypothesis testing"
- [discussion] Bugs related to the implementation of p.adjust() were fixed across Normalization/Correction/Statistics scripts: "Bugs related to the implementation of p.adjust() function has been fixed throughout 7.Normalization/4.Correction/9.Statistics scripts."
- [intro] OUKS provides a comprehensive nine-step LC-MS untargeted metabolomics processing toolbox within which Statistics is the final step: "providing comprehensive nine step LC-MS untargeted metabolomic profiling data processing toolbox"
- [readme] Runtime requirements for running the OUKS statistics scripts: "The only requirements are to be familiar with the basic syntax of the R language, PC with Internet connection and Windows OS (desirable)"
- [readme] Metadata table defining sample groups is provided alongside the datasets: "[`Metadata`](https://github.com/plyush1993/OUKS/blob/main/Datasets%20(csv)/metadata.csv) table is also provided"
