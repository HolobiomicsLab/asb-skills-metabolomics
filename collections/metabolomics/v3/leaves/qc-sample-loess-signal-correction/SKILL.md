---
name: qc-sample-loess-signal-correction
description: Use when a metabolomics featuredata matrix contains QC samples analysed
  periodically throughout the batch (sampledata 'class' column = 0 for QCs) and intensity
  drift across run order must be corrected with a robust LOESS fit.
when_to_use_negative:
- The goal is normalization using internal/external standards or QC metabolites —
  that is handled by NormQcmets, not NormQcsamples.
- The desired normalization is simple scaling (median, mean, sum, ref) — use NormScaling
  instead of a QC-sample LOESS fit.
- Featuredata is already log-transformed — with the default lg=TRUE the function expects
  raw abundances and applies log transformation itself after correction.
edam_operation: http://edamontology.org/operation_3662
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: NormalizeMets
  role: Provides the NormQcsamples function implementing QC-RLSC with method='rlsc',
    span/deg/lg parameters, and saveoutput/outputname options
  repo: https://github.com/metabolomicstats/NormalizeMets
- name: statTarget
  role: Alternative implementation of the QC-RLSC signal correction method as described
    by Dunn et al. 2011
- name: R
  role: Runtime environment (version 3.4.3 or higher) in which NormalizeMets and its
    NormQcsamples function are executed
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:14d584fb350ab1297032c49c60f955e966f7b46dc45121de84b955a8a6e292a3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/metabolomicstats/NormalizeMets
  tools:
  - NormalizeMets
  - R
  - statTarget
  edam_operation: http://edamontology.org/operation_3662
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# qc-sample-loess-signal-correction

## Summary

Applies QC-RLSC (quality control sample based robust LOESS signal correction, per Dunn et al. 2011) via the NormalizeMets NormQcsamples function with method='rlsc', to remove drift-induced unwanted variation in metabolomics feature intensities across a run order.

## When to use

Use when a metabolomics featuredata matrix contains QC samples analysed periodically throughout the batch (sampledata 'class' column = 0 for QCs) and intensity drift across run order must be corrected with a robust LOESS fit.

## When NOT to use

- The goal is normalization using internal/external standards or QC metabolites — that is handled by NormQcmets, not NormQcsamples.
- The desired normalization is simple scaling (median, mean, sum, ref) — use NormScaling instead of a QC-sample LOESS fit.
- Featuredata is already log-transformed — with the default lg=TRUE the function expects raw abundances and applies log transformation itself after correction.

## Inputs

- featuredata matrix: metabolite peak intensities with unique sample names as row names and unique metabolite names as column names
- sampledata dataframe with 'batch', 'class' and 'order' columns, row names = unique sample names, QC samples marked class 0

## Outputs

- QC-RLSC corrected (and by default log-transformed) featuredata matrix of the same dimensions as the ordered input
- optionally a CSV file (e.g. normalized_qcsamples.csv) when saveoutput=TRUE

## How to apply

Prepare a sampledata frame whose columns include 'batch', 'class' and 'order', with QC samples assigned class 0, then order both featuredata and sampledata by the run order column. Call NormQcsamples on the untransformed (raw) featuredata with method='rlsc'; the function fits a robust LOESS signal correction per feature across the QC samples, with tunable parameters span (default 0), deg (default 2), and lg (default TRUE). Because the default lg=TRUE applies log transformation, run the function on raw abundances and let it log-transform after signal correction rather than pre-log-transforming the input. Set saveoutput=TRUE with an outputname to persist the corrected matrix, and verify the output has the same rows (samples) and columns (metabolites) as the ordered input with no missing entries.

## Related tools

- **NormalizeMets** (Provides the NormQcsamples function implementing QC-RLSC with method='rlsc', span/deg/lg parameters, and saveoutput/outputname options) — https://github.com/metabolomicstats/NormalizeMets
- **statTarget** (Alternative implementation of the QC-RLSC signal correction method as described by Dunn et al. 2011)
- **R** (Runtime environment (version 3.4.3 or higher) in which NormalizeMets and its NormQcsamples function are executed) — https://cran.r-project.org/

## Examples

```
NormQcsamples(featuredata=ordered_featuredata, sampledata=ordered_sampledata, method="rlsc", span=0, deg=2, lg=TRUE, saveoutput=TRUE, outputname="normalized_qcsamples.csv")
```

## Evaluation signals

- Output matrix has the same number of rows (samples) and columns (metabolites) as the ordered Didata featuredata
- All entries of the corrected matrix are present (non-missing)
- QC samples in sampledata are correctly assigned class 0 and data ordered by the 'order' column before fitting
- Corrected data has been log-transformed when the default lg=TRUE was used (normalization applied before log transformation)
- Saved output file (saveoutput=TRUE) matches the supplied outputname (e.g. normalized_qcsamples.csv)

## Limitations

- Requires ordered run data: featuredata and sampledata must be sorted by run order before correction, and sampledata must contain 'batch', 'class' and 'order' columns with QCs at class 0.
- The default lg=TRUE means the function log-transforms corrected values — applying it to pre-log-transformed data would double-transform.
- LOESS behaviour depends on span and deg settings (defaults span=0, deg=2); non-default tuning may need reassessment against the dataset's research question as NormalizeMets advocates.

## Evidence

- [readme] The software requires the installation of R software (version 3.4.3 or higher).: [evidence span withheld — anchor status: external_doc]
- [readme] "featuredata" which is the metabolomics data matrix containing all metabolite peak intensities (or concentrations). Unique sample names must be provided as row names and unique metabolite names as column names: [evidence span withheld — anchor status: external_doc]
