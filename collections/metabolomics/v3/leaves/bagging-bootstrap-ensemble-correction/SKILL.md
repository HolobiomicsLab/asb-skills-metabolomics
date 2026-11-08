---
name: bagging-bootstrap-ensemble-correction
description: Use when a .csv metabolomics matrix (experiments in rows, features in
  columns, batch levels in the first column) shows across-batch signal drift and you
  want the ber-bagging ensemble correction or to compare it against the ber, parametric-ComBat,
  and nonparametric-ComBat outputs.
when_to_use_negative:
- 'Datasets with more than 2000 features: the README suggests Visodbnorm/dbnormSCORE-style
  functions only for less than 2000 features (variables) for better computational
  speed — prefer the decomposed dbnorm functions on subsets or other pipelines'
- Untargeted data needing unsupervised drift inspection only — hclustdbnorm (Pearson
  distance, average linkage) or raw-data PCA may suffice before committing to a correction
  model
- Data where the goal is metabolite annotation or pathway mapping (HMDB/KEGG) rather
  than removal of technical heterogeneity across batches
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: dbnorm
  role: R package providing dbnormBagging for ber-bagging batch-effect correction
    with partial bagging (n=150 bootstrap samples)
  repo: https://github.com/NBDZ/dbnorm
- name: ber
  role: CRAN dependency (installed via devtools::install_version("ber", version =
    "4.0")) providing the base two-stage L/S regression that bagging aggregates
- name: sva
  role: Bioconductor dependency providing the empirical-Bayes ComBat models that dbnormBagging
    output is compared against
- name: bootstrap
  role: CRAN dependency supporting the bagging/bootstrap resampling scheme
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:2f8cfbbff10730a274b3fe978153369c3d4308b0c136961f0cb7378126082efa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - R
  - ber
  - bootstrap
  - dbnorm
  - sva
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# bagging-bootstrap-ensemble-correction

## Summary

Applies the bagging (bootstrap-aggregated) variant of the ber two-stage location/scale regression from the dbnorm R package to remove drift across analytical batches in metabolomics feature matrices, offering an ensemble alternative to parametric ComBat, nonparametric ComBat, and plain ber.

## When to use

Use when a .csv metabolomics matrix (experiments in rows, features in columns, batch levels in the first column) shows across-batch signal drift and you want the ber-bagging ensemble correction or to compare it against the ber, parametric-ComBat, and nonparametric-ComBat outputs.

## When NOT to use

- Datasets with more than 2000 features: the README suggests Visodbnorm/dbnormSCORE-style functions only for less than 2000 features (variables) for better computational speed — prefer the decomposed dbnorm functions on subsets or other pipelines
- Untargeted data needing unsupervised drift inspection only — hclustdbnorm (Pearson distance, average linkage) or raw-data PCA may suffice before committing to a correction model
- Data where the goal is metabolite annotation or pathway mapping (HMDB/KEGG) rather than removal of technical heterogeneity across batches

## Inputs

- Normalized, log2-scaled metabolomics feature matrix in .csv format with independent experiments in rows and features (variables) in columns, with batch levels in the first column
- Optional: pre-imputed matrix from dbnorm's emvd (lowest detected value across the whole experiment) or emvf (per-feature lowest value) functions

## Outputs

- Batch-corrected data matrix exported as .csv in the R temporary folder (folder initiating with 'Rtmpe')
- PDF with PCA plot, Scree plot, and Correlation plot for the bagging-ber corrected data saved in the working directory
- Two-column matrix of adjusted-R² per variable quantifying residual batch dependence
- RLA plots visualized in the RStudio Viewer panel

## How to apply

Prepare the input via read.csv in R with header=TRUE and row.names=1, after dbnorm's log2 normalization/scaling guidance, keeping batch levels in the first column; optionally impute missing (Zero/NA) values first with emvd or emvf. Then call dbnormBagging(data), which runs the two-stage ber procedure (Giordan 2013, linear fitting of both location and scale parameters) aggregated over partial bagging with n=150 bootstrap samples, and writes PCA/Scree/Correlation PDF plots to the working directory, the corrected matrix to a .csv in the R temporary folder, the adjusted-R-squared matrix for each variable's dependency on batch, and RLA plots in the RStudio Viewer. Judge model performance with dbnormSCORE, whose 0-to-1 score is based on the maximum adjusted-R² explained by batch after correction. In the reference study on the SKIPOGH plasma LC-MS cohort (1,079 individuals), bagging-ber corrected data showed no residual association of metabolite intensities to batch level, performing alongside parametric ComBat and plain ber.

## Related tools

- **dbnorm** (R package providing dbnormBagging for ber-bagging batch-effect correction with partial bagging (n=150 bootstrap samples)) — https://github.com/NBDZ/dbnorm
- **ber** (CRAN dependency (installed via devtools::install_version("ber", version = "4.0")) providing the base two-stage L/S regression that bagging aggregates)
- **sva** (Bioconductor dependency providing the empirical-Bayes ComBat models that dbnormBagging output is compared against)
- **bootstrap** (CRAN dependency supporting the bagging/bootstrap resampling scheme)

## Examples

```
library(dbnorm); corrected <- dbnormBagging(data)
```

## Evaluation signals

- dbnormSCORE adjusted-R² analysis: bagging-ber corrected data should show metabolite intensities with no association to batch level (as reported alongside parametric ComBat and ber, unlike lowess or nonparametric ComBat where residual association remained)
- PCA/Scree plot PDF of corrected data should no longer show samples clustered by batch order, unlike the raw-data PCA in the study where samples clustered by batch
- Corrected matrix .csv is written to the Rtmpe temporary folder and the plotting PDF to the working directory — verify both artifacts exist
- Adjusted-R² two-column matrix returned per variable should reflect maximum variability explained by batch near zero for well-corrected features (ber and parametric ComBat reached < 0.01, i.e. 1%, in SKIPOGH)
- Bootstrap parameter: the run must use partial bagging with n=150 bootstrap samples as specified for the bagging model

## Limitations

- Precision of the bagging implementation depends on correct installation of archived CRAN dependencies (ber 4.0, NormalizeMets 0.25, metabolomics 0.1.4) via devtools::install_version; dbnorm itself is distributed as a tar.gz from GitHub (v0.2.2), not CRAN
- Input data must already be normalized and scaled on log2 to account for high-abundance features, otherwise technical heterogeneity from abundant variables may be overlooked
- Model choice is data-dependent: in the SKIPOGH study, nonparametric ComBat and lowess left residual batch variation (up to 0.60 and 0.78), so bagging-ber's good performance there does not guarantee it for every dataset — compare via dbnormSCORE
- Missing values (Zero or NA) should be handled first via emvd/emvf before correction

## Evidence

- [readme] Bagging model is performed using partial bagging with n=150 bootstrap samples: "Bagging model is performed using partial bagging with n=150 bootstrap samples"
- [abstract] ber removes batch effects by linear fitting for both location and scale (L/S) parameters: "ber, which removes batch effects by using linear fitting for both location and scale (L/S) parameters"
- [abstract] No association remained in parametric ComBat-, ber- and bagging ber-corrected data: "no association was present in parametric Com Bat-, ber- and bagging ber-corrected data."
- [methods] dbnormBer, dbnormBagging, dbnormPcom, dbnormNPcom normalize across-batch signal drift: "Data normalization for across batches signal drift using either of ber-,ber-bagging, parametric ComBat- and non parametric ComBat- models and unsupervised clustering and regression analysis of"
- [results] ber and parametric ComBat corrected datasets showed maximum batch variability < 0.01: "the datasets corrected by ber and parametric Com Bat functions presented a similar performance in this study, with a maximum variability associated with the batch < 0.01 (1%)"
