---
name: metabolomics-matrix-normalization
description: Use when a log-transformed, missing-value-imputed metabolomics peak-intensity
  matrix (featuredata format) needs sample-wise scaling normalization, or when comparing
  sum/median/mean/reference scaling variants before downstream differential or classification
  analysis.
when_to_use_negative:
- The data are raw and not yet log-transformed and imputed — apply LogTransform and
  MissingValues first, since the package workflow requires an analysis-ready matrix
- The unwanted variation is driven by batch/matrix effects or QC drift better addressed
  by the package's other functions (NormQcmets using internal/external standards,
  or NormQcsamples QC-RLSC signal correction) rather than simple per-sample scaling
- A combined sequential normalization is needed — use NormCombined instead of a single
  scaling method
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NormalizeMets
  role: R package providing the NormScaling function (methods median/mean/sum/ref)
    plus LogTransform and MissingValues preprocessing functions used in this scaling-normalization
    workflow
  repo: github.com__metabolomicstats__NormalizeMets
- name: R
  role: Runtime environment (version 3.4.3 or higher) in which the NormalizeMets functions
    are executed
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:9caae6014749869b8802514a0b6b1a1e5fa8a53c5b4a811c6599d2d9b7c99ba5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NormalizeMets
  - R
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metabolomics-matrix-normalization

## Summary

Apply scaling normalization to a metabolomics featuredata matrix (metabolites in columns, unique sample names as rows) using the NormalizeMets package's NormScaling function, which offers sum, median, mean, and reference-vector scaling. Preprocessing via LogTransform(zerotona=TRUE) and MissingValues(method="knn") is required first to obtain an analysis-ready matrix.

## When to use

Use when a log-transformed, missing-value-imputed metabolomics peak-intensity matrix (featuredata format) needs sample-wise scaling normalization, or when comparing sum/median/mean/reference scaling variants before downstream differential or classification analysis.

## When NOT to use

- The data are raw and not yet log-transformed and imputed — apply LogTransform and MissingValues first, since the package workflow requires an analysis-ready matrix
- The unwanted variation is driven by batch/matrix effects or QC drift better addressed by the package's other functions (NormQcmets using internal/external standards, or NormQcsamples QC-RLSC signal correction) rather than simple per-sample scaling
- A combined sequential normalization is needed — use NormCombined instead of a single scaling method

## Inputs

- featuredata matrix: metabolite peak intensities (or concentrations), metabolites in columns, unique sample names as row names
- reference vector (refvec) such as sample weight or volume, required only for method="ref"
- optionally sampledata (sample-specific information) and metabolitedata (metabolite-specific annotations) per the package input format

## Outputs

- Normalized featuredata matrix returned in the NormScaling output object, with the same row (sample) and column (metabolite) names as the input
- CSV file written to disk when saveoutput=TRUE with a named outputname

## How to apply

First load the alldata_eg example (or your own) featuredata matrix — metabolite peak intensities in columns, unique sample names as row names — then log-transform with LogTransform(zerotona=TRUE) and impute missing values with MissingValues(method="knn", feature.cutoff=0.8, sample.cutoff=0.8, k=10), giving imp$featuredata. Run NormScaling on imp$featuredata choosing one of method=c("median", "mean", "sum", "ref"); median scaling uses the median of each sample's intensities, while method="ref" additionally requires a reference vector passed via refvec (e.g. sample weight or volume) to which metabolite abundances are normalized. Set saveoutput=TRUE with a named outputname to write the normalized featuredata matrix as a CSV file. Validate that the returned output object contains a normalized featuredata matrix whose rows (samples) and columns (metabolites) match the input matrix exactly.

## Related tools

- **NormalizeMets** (R package providing the NormScaling function (methods median/mean/sum/ref) plus LogTransform and MissingValues preprocessing functions used in this scaling-normalization workflow) — github.com__metabolomicstats__NormalizeMets
- **R** (Runtime environment (version 3.4.3 or higher) in which the NormalizeMets functions are executed) — https://cran.r-project.org/

## Examples

```
imp <- MissingValues(LogTransform(featuredata, zerotona=TRUE)$featuredata, method="knn", feature.cutoff=0.8, sample.cutoff=0.8, k=10); out <- NormScaling(imp$featuredata, method="median", saveoutput=TRUE, outputname="norm.results")
```

## Evaluation signals

- NormScaling output object contains a normalized featuredata matrix (not NULL) after the call
- Saved CSV (saveoutput=TRUE with outputname) exists and its row names (samples) and column names (metabolites) are identical to the input feature matrix
- All four variants can be reproduced: method="sum", "median", "mean" run without refvec; method="ref" runs only when refvec is supplied
- Input passed to NormScaling is imp$featuredata from LogTransform + MissingValues with feature.cutoff=0.8, sample.cutoff=0.8, k=10
- Sample-wise statistics (sum, median, or mean of intensities per sample) are constant across samples in the normalized matrix, matching the chosen scaling method

## Limitations

- Scaling normalizations address overall sample intensity differences only; the README notes metabolomics data are also subject to unwanted variation from batch effects, matrix effects, and confounding biological variation, which scaling alone does not remove
- method="ref" requires a meaningful external reference vector (e.g. sample weight or volume); without refvec the ref method cannot be applied
- Rows must carry unique sample names and columns unique metabolite names per the required featuredata format

## Evidence

- [methods] Scaling normalization method description: [evidence span withheld — anchor status: not_found]
- [readme] Required featuredata input format: [evidence span withheld — anchor status: external_doc]
- [readme] R version requirement: [evidence span withheld — anchor status: external_doc]
