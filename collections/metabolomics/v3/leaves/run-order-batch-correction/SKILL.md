---
name: run-order-batch-correction
description: Use when a metabolomics featuredata matrix was acquired with QC samples
  (class 0 in sampledata) injected periodically across the batch, run order is recorded,
  and instrument drift should be corrected with robust LOESS (method='rlsc') before
  log transformation.
when_to_use_negative:
- QC samples were not injected periodically throughout the batch, or sampledata lacks
  'batch', 'class' and 'order' columns with QC samples assigned class 0
- The goal is normalization using quality control metabolites (internal/external standards),
  which calls NormQcmets with methods like 'is', 'ccmn', or 'ruv2' instead
- The data are already log-transformed, since the default lg=TRUE applies normalization
  before log transformation and expects raw featuredata
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3408
tools:
- name: NormalizeMets
  role: R package providing NormQcsamples with method='rlsc' that implements the QC-RLSC
    robust LOESS signal correction
  repo: metabolomicstats/NormalizeMets
- name: statTarget
  role: Alternative R implementation of the QC-RLSC method referenced by the article
    (Luan et al. 2017)
- name: R
  role: Runtime environment required (version 3.4.3 or higher) for running NormQcsamples
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:fbe8463d3319d9a705b1885bdcb62908ac6cd2f8309c18ba2af5fb24829c1d15
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/metabolomicstats/NormalizeMets
  tools:
  - NormalizeMets
  - R
  - statTarget
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3408
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# run-order-batch-correction

## Summary

Applies QC-RLSC (quality control sample based robust LOESS signal correction, Dunn et al. 2011) to a metabolomics feature matrix via the NormalizeMets NormQcsamples function, correcting signal drift across acquisition run order using periodically injected QC samples.

## When to use

Use when a metabolomics featuredata matrix was acquired with QC samples (class 0 in sampledata) injected periodically across the batch, run order is recorded, and instrument drift should be corrected with robust LOESS (method='rlsc') before log transformation.

## When NOT to use

- QC samples were not injected periodically throughout the batch, or sampledata lacks 'batch', 'class' and 'order' columns with QC samples assigned class 0
- The goal is normalization using quality control metabolites (internal/external standards), which calls NormQcmets with methods like 'is', 'ccmn', or 'ruv2' instead
- The data are already log-transformed, since the default lg=TRUE applies normalization before log transformation and expects raw featuredata

## Inputs

- featuredata: metabolomics data matrix of peak intensities with unique sample names as row names and unique metabolite names as column names (untransformed/raw)
- sampledata: dataframe of sample-specific information with 'batch', 'class' and 'order' columns, unique sample names as row names, QC samples labeled class 0

## Outputs

- QC-RLSC signal-corrected (and log-transformed when lg=TRUE) feature data matrix with the same samples x metabolites dimensions as the ordered input
- Optional CSV output file, e.g. normalized_qcsamples.csv

## How to apply

Prepare the sampledata frame so it contains the required 'batch', 'class' and 'order' columns, with QC samples assigned class 0, and order both the featuredata matrix and sampledata by the run-order column. Run the NormalizeMets function NormQcsamples on the untransformed (raw) featuredata with method='rlsc' so a robust LOESS signal correction is fitted per feature across the QC samples, using the default span=0 and polynomial degree deg=2. Since the default lg=TRUE, the function applies log transformation after normalization, so input data must be raw, not pre-log-transformed. Set saveoutput=TRUE and outputname (e.g. 'normalized_qcsamples') to persist the corrected matrix. The QC-RLSC approach is the same method described by Dunn et al. 2011 and implemented in statTarget.

## Related tools

- **NormalizeMets** (R package providing NormQcsamples with method='rlsc' that implements the QC-RLSC robust LOESS signal correction) — metabolomicstats/NormalizeMets
- **statTarget** (Alternative R implementation of the QC-RLSC method referenced by the article (Luan et al. 2017))
- **R** (Runtime environment required (version 3.4.3 or higher) for running NormQcsamples) — https://cran.r-project.org/

## Examples

```
NormQcsamples(featuredata=Didata_f, sampledata=Didata_s[order(Didata_s$order),], method="rlsc", span=0, deg=2, lg=TRUE, saveoutput=TRUE, outputname="normalized_qcsamples")
```

## Evaluation signals

- Output matrix has the same number of rows (samples) and columns (metabolites) as the ordered input featuredata (e.g. the Didata example)
- All entries in the output matrix are present and non-missing
- sampledata satisfied the schema requirement: 'batch', 'class', 'order' columns present with QC samples assigned class 0, and both matrices ordered by the 'order' column
- Function call used method='rlsc' on untransformed featuredata with documented defaults span=0, deg=2, lg=TRUE
- saveoutput=TRUE produced a CSV with the specified outputname (e.g. normalized_qcsamples)

## Limitations

- The method argument is fixed to 'rlsc' in NormQcsamples; only the QC-sample-based robust LOESS correction path is available
- Requires sample metadata including run order and QC class labels; correction quality depends on periodic QC injections across the batch
- Unwanted variation stems from batch effects, matrix effects and confounding biological variation, so QC-RLSC alone may not address all normalization goals; the package recommends comparing methods per dataset

## Evidence

- [readme] featuredata input format requirement: [evidence span withheld — anchor status: external_doc]
- [readme] sampledata input format requirement: [evidence span withheld — anchor status: external_doc]
