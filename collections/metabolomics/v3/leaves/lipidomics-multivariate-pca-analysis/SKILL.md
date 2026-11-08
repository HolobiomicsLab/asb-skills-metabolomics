---
name: lipidomics-multivariate-pca-analysis
description: Use when a lipidomics dataset (Skyline export, numerical matrix, or Metabolomics
  Workbench study such as ST001111) has been imported as a LipidomicsExperiment and
  you need PCA scores plots for quality control, outlier detection, or pattern discovery.
when_to_use_negative:
- The analysis goal is supervised class separation between defined groups (e.g. Benign
  vs Cancer loadings ranking) — use lipidr's OPLS-DA method instead of unsupervised
  PCA
- The data are not lipid peak areas / lipidomics measurements and lipid class/chain
  inference from lipid names is not applicable
- Samples have not yet been imported into a LipidomicsExperiment with sample annotations,
  so group coloring and outlier interpretation are impossible
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3360
tools:
- name: lipidr
  role: R/Bioconductor package providing mva(method="PCA") on LipidomicsExperiment
    objects with interactive scores/loadings plots for quality control and outlier
    detection
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Runtime environment; LipidomicsExperiment extends SummarizedExperiment to
    integrate with other Bioconductor packages
- name: Metabolomics Workbench API
  role: Used to search and download public lipidomics studies (e.g. ST001111) directly
    into lipidr for reanalysis
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:647ab3fae2d8fbec47308f04ce83305d9925db25ccb6d85733154da7f34e65b0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Metabolomics Workbench API
  - R
  - R/Bioconductor
  - lipidr
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# lipidomics-multivariate-pca-analysis

## Summary

Perform principal component analysis (PCA) on lipidomics peak-area data using lipidr's multivariate analysis workflow to reveal sample patterns and flag outlier samples (e.g. those with significantly large dispersion) for removal before downstream analysis.

## When to use

Use when a lipidomics dataset (Skyline export, numerical matrix, or Metabolomics Workbench study such as ST001111) has been imported as a LipidomicsExperiment and you need PCA scores plots for quality control, outlier detection, or pattern discovery.

## When NOT to use

- The analysis goal is supervised class separation between defined groups (e.g. Benign vs Cancer loadings ranking) — use lipidr's OPLS-DA method instead of unsupervised PCA
- The data are not lipid peak areas / lipidomics measurements and lipid class/chain inference from lipid names is not applicable
- Samples have not yet been imported into a LipidomicsExperiment with sample annotations, so group coloring and outlier interpretation are impossible

## Inputs

- LipidomicsExperiment object built from a Skyline CSV export (read_skyline) or a numerical matrix (lipids as rows, samples as columns) plus a sample-annotation table with identical sample names
- Metabolomics Workbench study files (e.g. ST001111) downloaded via the API integration
- Sample annotation / clinical groups table (sample names in first column, e.g. SampleType)

## Outputs

- Interactive PCA scores plot (points colored by sample group, e.g. SampleType) saved as figure/*.png
- List of outlier samples with significantly large dispersion to remove from the dataset
- Cleaned LipidomicsExperiment (e.g. saved as cleaned_st001111.rds) for downstream univariate/multivariate analysis

## How to apply

Load the lipidomics data into lipidr's LipidomicsExperiment container (via read_skyline, as_lipidomics_experiment, or the Metabolomics Workbench API), attaching sample annotations with add_sample_annotation. Run the multivariate analysis with mva(measure="Area", method="PCA") so PCA is computed on lipid peak areas, and plot the interactive scores plot with points colored by a clinical/group column such as SampleType. Inspect the scores plot for samples with significantly large dispersion from the main cluster — in the ST001111 breast-cancer example, samples 42 and 18 were flagged this way — and remove them before further analysis. PCA in lipidr is one of several multivariate methods alongside PCoA and OPLS-DA, so reserve PCA for unsupervised quality control/pattern discovery and use OPLS-DA when a two-class outcome (e.g. Benign vs Cancer) is the target.

## Related tools

- **lipidr** (R/Bioconductor package providing mva(method="PCA") on LipidomicsExperiment objects with interactive scores/loadings plots for quality control and outlier detection) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Runtime environment; LipidomicsExperiment extends SummarizedExperiment to integrate with other Bioconductor packages)
- **Metabolomics Workbench API** (Used to search and download public lipidomics studies (e.g. ST001111) directly into lipidr for reanalysis)

## Examples

```
d <- read_skyline("Skyline_export.csv"); d <- add_sample_annotation(d, "clin.csv"); mva(d, measure="Area", method="PCA")
```

## Evaluation signals

- PCA is computed on the 'Area' measure via mva(measure="Area", method="PCA") rather than another intensity column
- The scores plot renders interactively with points colored by the loaded group column (SampleType in ST001111)
- Outlier samples with significantly large dispersion are identified visually — in the ST001111 reproduction this yields exactly samples 42 and 18
- Removed outliers produce a cleaned dataset saved as RDS (e.g. cleaned_st001111.rds) and a saved PCA figure under figure/*.png
- The remaining samples cluster by SampleType with no extreme-dispersion points left in the scores plot

## Limitations

- PCA is unsupervised; it reveals patterns but cannot rank lipids by class-separation contribution the way OPLS-DA loadings (plot_mva_loadings / top_lipids) can
- Outlier identification relies on visual inspection of the interactive scores plot; no automatic numeric dispersion threshold is specified in the article or README
- lipidr requires sample names in the annotation table to be identical to those in the data matrix, otherwise add_sample_annotation fails

## Evidence

- [readme] `lipidr` implements PCA, PCoA and OPLS(DA) to reveal patterns in data and discover variables related to an outcome of interest.: [evidence span withheld — anchor status: external_doc]
