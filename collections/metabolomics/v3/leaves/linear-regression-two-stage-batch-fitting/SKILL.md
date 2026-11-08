---
name: linear-regression-two-stage-batch-fitting
description: Use when a log2-scaled LC-MS metabolomics matrix (.csv, samples in rows,
  features in columns, batch levels in the first column) shows across-batch signal
  drift — e.
when_to_use_negative:
- The dataset has more than ~2000 features and you need interactive visualization
  — dbnormBer is suggested for less than 2000 features for better computational speed;
  decompose or subsample instead
- The goal is differential-abundance testing (e.g. volcano plots with Log2FC > 1.5,
  adj-p < 0.05) or pathway enrichment (MSEA) rather than batch-effect removal
- Data are not yet missing-value imputed — run emvd/emvf (lowest detected value, whole
  experiment or per feature/column) before batch correction
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: dbnorm
  role: R package providing dbnormBer and dbnormBagging functions that implement the
    ber two-stage linear-regression L/S batch correction and its bagging variant
  repo: https://github.com/NBDZ/dbnorm
- name: ber
  role: CRAN package whose two-stage procedure (Giordan 2013, DOI:10.1007/s12561-013-9081-1)
    supplies the underlying linear location/scale batch-effect model; installed at
    version 4.0 from the CRAN archive
  repo: https://cran.r-project.org/src/contrib/Archive/
- name: R
  role: Runtime environment; dependencies installed from CRAN and Bioconductor (sva,
    limma, pcaMethods, etc.)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:26b27d968a99aebf61a7ccfe68be756e9bf3f096876160e25c86220836269043
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# linear-regression-two-stage-batch-fitting

## Summary

Applies the ber two-stage linear-regression batch-effect model (and its ber-bagging variant, partial bagging with n=150 bootstrap samples), as implemented in the dbnorm R package, to fit and remove batch-associated location and scale (L/S) parameters from a batch-annotated metabolomics matrix.

## When to use

Use when a log2-scaled LC-MS metabolomics matrix (.csv, samples in rows, features in columns, batch levels in the first column) shows across-batch signal drift — e.g. PCA clustering by batch order or >50% of metabolite variability explained by batch — and a location/scale regression correction is required.

## When NOT to use

- The dataset has more than ~2000 features and you need interactive visualization — dbnormBer is suggested for less than 2000 features for better computational speed; decompose or subsample instead
- The goal is differential-abundance testing (e.g. volcano plots with Log2FC > 1.5, adj-p < 0.05) or pathway enrichment (MSEA) rather than batch-effect removal
- Data are not yet missing-value imputed — run emvd/emvf (lowest detected value, whole experiment or per feature/column) before batch correction

## Inputs

- Batch-annotated metabolomics matrix in .csv format, independent experiments in rows, features (variables) in columns, batch levels in the first column, normalized and log2-scaled
- Optional: QC/analytical replicate samples present in different batches for hclustdbnorm validation

## Outputs

- Batch-corrected data matrix exported as .csv in a temporary folder (Rtmpe in the user's Temporary directory)
- PDF with PCA plot, Scree plot and Correlation plot saved in the working directory
- Two-column matrix of adjusted-R2 values per variable for raw vs corrected data
- RLA plots rendered in the RStudio Viewer panel

## How to apply

Load the normalized, log2-scaled .csv matrix with read.csv (header=T, row.names=1), keeping the batch level column. Run dbnormBer(data) to fit the two-stage linear-regression model of Giordan (2013) (DOI:10.1007/s12561-013-9081-1), originally developed for microarray data, which removes batch effects by linear fitting of both location and scale (L/S) parameters; outputs include a PCA/Scree/Correlation PDF and a corrected-data .csv. Optionally run dbnormBagging(data) for the bagging variant, which uses partial bagging with 150 bootstrap samples to stabilize the fit. Judge correctness via dbnormSCORE: ber-corrected data should show maximum adjusted-R2 residual batch variability < 0.01, as reported for ber and parametric ComBat in the SKIPOGH dataset, unlike lowess (up to 0.78) or nonparametric ComBat (up to 0.60).

## Related tools

- **dbnorm** (R package providing dbnormBer and dbnormBagging functions that implement the ber two-stage linear-regression L/S batch correction and its bagging variant) — https://github.com/NBDZ/dbnorm
- **ber** (CRAN package whose two-stage procedure (Giordan 2013, DOI:10.1007/s12561-013-9081-1) supplies the underlying linear location/scale batch-effect model; installed at version 4.0 from the CRAN archive) — https://cran.r-project.org/src/contrib/Archive/
- **R** (Runtime environment; dependencies installed from CRAN and Bioconductor (sva, limma, pcaMethods, etc.))

## Examples

```
library(dbnorm); data <- read.csv("path/to/mydata.csv", sep = ",", header = T, row.names = 1); dbnormBer(data); dbnormBagging(data)
```

## Evaluation signals

- After correction, per-metabolite adjusted-R2 regression on batch level shows maximum residual variability < 0.01 (ber matched parametric ComBat in SKIPOGH, where >50% of variability was batch-explained pre-correction)
- dbnormSCORE returns a score near 1 (0–1 scale based on highest adjusted-R2), indicating ber consistently accommodated technical variability across all detected features
- PCA/scree PDF of corrected data shows samples no longer clustered by batch order (raw SKIPOGH PCA clustered samples by batch)
- Corrected .csv matrices and PDF diagnostics are actually written to the Rtmpe temporary folder and working directory, confirming the run completed
- Biological signal preserved: correlation between LC-MS creatinine and clinically measured creatinine increases post-correction (raw r=0.18, model-corrected ~0.61)

## Limitations

- Bagging variant uses partial bagging with n=150 bootstrap samples — computational cost scales with feature count; all dbnorm correction functions are suggested for fewer than 2000 features
- ber-corrected output was not uniquely best for all metrics: lowess-corrected data improved LC-MS/MS vs clinical creatinine correlation to 0.5 while model-corrected data reached 0.61, but lowess filtered out some metabolites (e.g. xanthosine 5′-monophosphate) — model choice requires dbnormSCORE comparison per dataset
- Input must already be normalized and log2-scaled; unlogged high-abundance features can let technical heterogeneity be overlooked

## Evidence

- [abstract] Another statistical model that was originally developed for microarray data is ber, which removes batch effects by using linear fitting for both location and scale (L/S) parameters: [evidence span withheld — anchor status: not_found]
- [methods] dbnormBer definition: "Data normalization for across batches signal drift using either of ber-"
- [readme] README bagging parameter: "Bagging model is performed using partial bagging with n=150 bootstrap samples"
- [readme] README input format: "The input data must be in **.csv** format with the independent experiments in the rows and the features (variables) in the columns, with the `batch` levels considered in the first column."
- [results] ber performance: "the datasets corrected by ber and parametric Com Bat functions presented a similar performance in this study, with a maximum variability associated with the batch < 0.01 (1%)"
