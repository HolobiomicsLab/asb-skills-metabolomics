---
name: r-package-data-normalization
description: Use when a metabolomics feature matrix (.csv, samples in rows, features
  in columns, batch levels in first column) shows samples clustering by batch (e.g.
  PCA groups by run order) and across-batch signal drift must be corrected and compared
  across statistical models.
when_to_use_negative:
- The dataset has fewer features than the models need to estimate batch parameters
  reliably, or the input is not batch-annotated (no batch levels in the first column)
  — ComBat/ber cannot estimate L/S batch parameters without batch labels
- The goal is differential-analysis statistics (volcano plots, Log2FC > 1.5, adj-p
  < 0.05) or pathway enrichment (MSEA) rather than batch-effect correction — those
  are downstream analyses of the corrected matrices
- Missing-value handling is the bottleneck — use the dbnorm preprocessing functions
  emvd/emvf (imputation by lowest detected value) instead of the correction models
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: dbnorm
  role: Core R package implementing Visodbnorm, dbnormBer, dbnormBagging, dbnormPcom,
    dbnormNPcom, dbnormSCORE, and hclustdbnorm for batch drift correction and model
    comparison
  repo: https://github.com/NBDZ/dbnorm
- name: sva
  role: Bioconductor dependency providing the empirical-Bayes ComBat framework underlying
    dbnorm's parametric and nonparametric ComBat corrections
- name: ber
  role: CRAN dependency (version 4.0 pinned via devtools::install_version) providing
    the two-stage linear location/scale batch correction model
- name: R
  role: Runtime environment; dbnorm is installed from GitHub as a tar.gz via R CMD
    INSTALL or devtools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:042022c8814508cf33bfed0a15ff853c8fff96788013ef6ad51955478651c1d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - R
  - ber
  - dbnorm
  - sva
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# r-package-data-normalization

## Summary

Batch-effect correction of batch-annotated metabolomics matrices in R via the dbnorm package, which applies parametric ComBat, nonparametric ComBat, ber, and bagging-ber (n=150 partial bagging) models to remove across-batch signal drift and score model fit with adjusted-R2.

## When to use

Use when a metabolomics feature matrix (.csv, samples in rows, features in columns, batch levels in first column) shows samples clustering by batch (e.g. PCA groups by run order) and across-batch signal drift must be corrected and compared across statistical models.

## When NOT to use

- The dataset has fewer features than the models need to estimate batch parameters reliably, or the input is not batch-annotated (no batch levels in the first column) — ComBat/ber cannot estimate L/S batch parameters without batch labels
- The goal is differential-analysis statistics (volcano plots, Log2FC > 1.5, adj-p < 0.05) or pathway enrichment (MSEA) rather than batch-effect correction — those are downstream analyses of the corrected matrices
- Missing-value handling is the bottleneck — use the dbnorm preprocessing functions emvd/emvf (imputation by lowest detected value) instead of the correction models

## Inputs

- Batch-annotated metabolomics matrix in .csv format (independent experiments in rows, features in columns, batch levels in first column, log2-normalized and scaled)
- Optionally raw metabolomics feature tables from LC-MS/MS processing (e.g. XCMS output) with QC/analytical replicate annotations for hclustdbnorm validation

## Outputs

- Four corrected data matrices (ber, ber-bagging, parametric ComBat, nonparametric ComBat) exported as .csv files in a temporary Rtmpe folder
- PDF diagnostic plots: PCA plot, Scree plot, and Correlation plot per correction model
- RLA plots rendered in the RStudio Viewer panel
- Two-column matrix of Adjusted R-squared per feature per model and a score table (dbnormSCORE) ranking the models

## How to apply

Load the batch-annotated matrix with read.csv (log2-scaled, batch levels in the first column) and call Visodbnorm(data) to run all three correction models—two-stage linear L/S fitting (ber, Giordan 2013), parametric and nonparametric empirical-Bayes ComBat (Johnson et al. 2007, sva package)—plus PCA/Scree/RLA diagnostics in one pass. For large data, decompose into dbnormBer, dbnormBagging (ber-bagging with partial bagging, n=150 bootstrap samples), dbnormPcom, and dbnormNPcom to export each corrected matrix separately; these functions are recommended over Visodbnorm for big data and each writes corrected .csv files plus PDF PCA/Scree/Correlation plots. Run dbnormSCORE(data) to compute adjusted-R2 (regression of each feature on batch level, raw vs corrected) and a 0–1 score based on the maximum variability explained by batch, then pick the model whose score is lowest/most consistent across features. Validate the chosen correction with hclustdbnorm (Pearson distance, average linkage) checking whether identical QC/analytical replicates from different batches cluster together after correction. Ground the choice biologically, e.g. by confirming metabolite intensities no longer associate with batch level; in the SKIPOGH study ber and parametric ComBat achieved residual batch variability <0.01 (1%) while nonparametric ComBat left up to 0.60 residual variation.

## Related tools

- **dbnorm** (Core R package implementing Visodbnorm, dbnormBer, dbnormBagging, dbnormPcom, dbnormNPcom, dbnormSCORE, and hclustdbnorm for batch drift correction and model comparison) — https://github.com/NBDZ/dbnorm
- **sva** (Bioconductor dependency providing the empirical-Bayes ComBat framework underlying dbnorm's parametric and nonparametric ComBat corrections)
- **ber** (CRAN dependency (version 4.0 pinned via devtools::install_version) providing the two-stage linear location/scale batch correction model)
- **R** (Runtime environment; dbnorm is installed from GitHub as a tar.gz via R CMD INSTALL or devtools)

## Examples

```
data <- read.csv("mydata.csv", header = TRUE, row.names = 1); library(dbnorm); Visodbnorm(data); dbnormSCORE(data)
```

## Evaluation signals

- Adjusted-R2 of feature-on-batch regressions drops to near 0 after correction; ber and parametric ComBat should reach maximum residual batch variability < 0.01 (1%) as in SKIPOGH
- dbnormSCORE returns a 0–1 score based on the maximum variability explained by batch; the chosen model should have the lowest/most consistent score across all features
- PCA of corrected data no longer clusters samples by batch order (raw SKIPOGH PCA clustered samples by batch; corrected data should not)
- hclustdbnorm dendrograms show identical QC/analytical replicates from different batches clustering together after correction (Pearson distance, average linkage)
- Four corrected .csv matrices plus PCA/Scree/Correlation PDF plots are written (working directory for PDFs, Rtmpe temp folder for CSVs), confirming all models ran

## Limitations

- Visodbnorm and dbnormSCORE are suggested for fewer than 2000 features for computational speed; use the decomposed dbnormBer/dbnormBagging/dbnormPcom/dbnormNPcom functions for big data
- Nonparametric ComBat and lowess corrections left substantial residual batch association (max residual variation 0.60 and 0.78 respectively in SKIPOGH); some metabolites may remain batch-confounded and lowess filtered out xanthosine 5′-monophosphate
- Input must be log2-normalized and scaled beforehand; raw unscaled high-abundance features can let technical heterogeneity be overlooked
- dbnorm version details (V-0.2.2) and exact ComBat/ber version pinning vary; ber 4.0, NormalizeMets 0.25, and metabolomics 0.1.4 must be installed from the CRAN archive
- Corrected .csv outputs land in the OS temporary directory (Rtmpe folder), not the working directory, and can be lost if not copied

## Evidence

- [methods] Visodbnorm: "This function performs batch effect adjustment via three statistical models implemented in the *dbnorm*"
- [readme] README bagging: "Bagging model is performed using partial bagging with n=150 bootstrap samples"
- [readme] README input format: "The input data must be in **.csv** format with the independent experiments in the rows and the features (variables) in the columns, with the `batch` levels considered in the first column."
- [readme] README score: "the performance of applied models are presented by a score calculated based on the maximum variability explained by the batch level"
- [results] results model comparison: "the datasets corrected by ber and parametric Com Bat functions presented a similar performance in this study, with a maximum variability associated with the batch < 0.01 (1%)"
- [abstract] abstract residual association: [evidence span withheld — anchor status: not_found]
