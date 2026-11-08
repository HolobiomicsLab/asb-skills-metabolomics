---
name: normalization-method-comparative-assessment
description: Use when a metabolomics featuredata matrix is subject to unwanted variation
  (batch/matrix effects) and several candidate normalization methods must be implemented,
  assessed, and compared before choosing one.
when_to_use_negative:
- Data are already normalized and the goal is only downstream analysis such as biomarker
  identification or clustering, not method comparison.
- The study design provides no QC samples, internal standards, or covariate information,
  so criteria needed to differentiate methods are unavailable.
- Only a single normalization method is mandated by the protocol and comparative evaluation
  is out of scope.
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: NormalizeMets
  role: Core R package implementing the normalization methods (NormQcmets, NormQcsamples,
    NormScaling, NormCombined) and their assessment
  repo: github.com__metabolomicstats__NormalizeMets
- name: R
  role: Runtime environment (version 3.4.3 or higher) in which the NormalizeMets functions
    are executed
  repo: https://cran.r-project.org/
- name: RStudio
  role: Recommended IDE for running the NormalizeMets workflow
- name: Microsoft Excel
  role: Graphical user interface (Excel 2016+, Windows 7+) exposing the same functions
    via ExNormalizeMets
- name: statTarget
  role: Implements the QC-RLSC signal correction underlying NormQcsamples
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:44c97c7e04e47d367af490c52271c94a383905b80d7bc745a5fdd4004bda225b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Microsoft Excel
  - NormalizeMets
  - R
  - RStudio
  - statTarget
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# normalization-method-comparative-assessment

## Summary

Comparatively evaluate metabolomics normalization methods (QC-metabolite, QC-sample QC-RLSC, scaling, and combined approaches) with the NormalizeMets R/Excel package, using dataset-dependent criteria to select the most suitable method for a given study.

## When to use

Use when a metabolomics featuredata matrix is subject to unwanted variation (batch/matrix effects) and several candidate normalization methods must be implemented, assessed, and compared before choosing one.

## When NOT to use

- Data are already normalized and the goal is only downstream analysis such as biomarker identification or clustering, not method comparison.
- The study design provides no QC samples, internal standards, or covariate information, so criteria needed to differentiate methods are unavailable.
- Only a single normalization method is mandated by the protocol and comparative evaluation is out of scope.

## Inputs

- featuredata matrix (metabolite peak intensities, unique sample names as row names, unique metabolite names as column names)
- metabolitedata dataframe with metabolite-specific IS/external-standard or control flags
- sampledata dataframe with sample type, order of analysis, and factors of interest (e.g. gender, Age, bmi)
- bundled example datasets: mixdata, Didata, UVdata, alldata_eg

## Outputs

- Normalized data matrices per method (writable via saveoutput/outputname)
- Combined-method (NormCombined) normalized matrix
- Assessment outputs: RLA plots, p-value distributions from LinearModelFit, PCA plots, heatmaps
- Comparison summary supporting selection of a normalization method

## How to apply

Organize input as featuredata (samples as rows, metabolites as columns), metabolitedata (internal/external standard flags), and sampledata (run order, factors). Preprocess by log transforming with LogTransform(zerotona=TRUE) and imputing missing values with MissingValues(method='knn', k=10, feature/sample cutoffs 0.8). Exercise the four method categories via dedicated functions: NormQcmets (methods 'nomis', 'ccmn', 'is', 'ruv2' with k=2 and a factormat of covariates such as gender, Age, bmi), NormQcsamples (QC-RLSC, method='rlsc', span=0, deg=2, lg=TRUE), NormScaling ('median', 'mean', 'sum', 'ref'), and NormCombined (e.g. c('rlsc','median')). Assess outputs with evaluation functions such as LinearModelFit residual RLA plots, p-value distributions, PcaPlots, and HeatMap, then judge methods by criteria appropriate to the dataset and research question before writing normalized matrices to CSV.

## Related tools

- **NormalizeMets** (Core R package implementing the normalization methods (NormQcmets, NormQcsamples, NormScaling, NormCombined) and their assessment) — github.com__metabolomicstats__NormalizeMets
- **R** (Runtime environment (version 3.4.3 or higher) in which the NormalizeMets functions are executed) — https://cran.r-project.org/
- **RStudio** (Recommended IDE for running the NormalizeMets workflow)
- **Microsoft Excel** (Graphical user interface (Excel 2016+, Windows 7+) exposing the same functions via ExNormalizeMets)
- **statTarget** (Implements the QC-RLSC signal correction underlying NormQcsamples)

## Examples

```
Rscript -e 'library(NormalizeMets); imp <- MissingValues(LogTransform(alldata_eg$featuredata, zerotona=TRUE)$featuredata, method="knn", k=10); fit <- LinearModelFit(featuredata=imp$featuredata, factormat=factormat, ruv2=FALSE)'
```

## Evaluation signals

- Each method family returns a normalized featuredata matrix with identical dimensions and row/column names to the log-transformed, imputed input.
- RLA plots of LinearModelFit residuals show reduced within-group variation for the selected method relative to alternatives.
- P-value distributions from LinearModelFit are interpretable and consistent with the factor structure supplied in factormat (e.g. gender, Age, bmi).
- NormCombined on c('rlsc','median') output differs from either single-method output, confirming both steps were applied.
- Normalized matrices are written to CSV (saveoutput=TRUE with outputname) and can be reloaded for downstream PcaPlots/HeatMap checks.

## Limitations

- Handles data locally; requires R >= 3.4.3 and, for the GUI, Microsoft Excel (2016) on Windows 7 or higher.
- Method choice is dataset- and question-dependent — no single method is universally best; comparative criteria must be selected per study.
- Missing value handling depends on the nature of missing data; MissingValues uses method='knn' or 'replace' with feature/sample cutoffs (default 0.8) that may need adjustment.
- Serves as an updated version of the earlier 'metabolomics' and 'MetNorm' R packages; older workflows may map differently.

## Evidence

- [readme] Input data format: [evidence span withheld — anchor status: external_doc]
