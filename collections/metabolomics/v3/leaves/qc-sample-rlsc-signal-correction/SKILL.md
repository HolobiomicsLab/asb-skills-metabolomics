---
name: qc-sample-rlsc-signal-correction
description: Use when metabolomics peak-intensity matrices (featuredata) show drift
  with run order and QC samples were injected periodically, so a QC-sample-based LOESS
  signal correction (rlsc) is needed before scaling or comparison.
when_to_use_negative:
- Normalization should be driven by internal/external standard metabolites or QC metabolites
  rather than QC samples — use NormQcmets (methods 'nomis', 'ccmn', 'is', 'ruv2')
  instead
- Only sample-wise median/mean/sum/reference scaling is needed with no run-order drift
  — use NormScaling rather than rlsc
- The dataset has no periodically injected QC samples, so QC-RLSC signal correction
  cannot be fit
edam_operation: http://edamontology.org/operation_0335
tools:
- name: NormalizeMets
  role: R package providing the NormQcsamples function that implements the QC-RLSC
    signal correction and NormCombined for chaining rlsc with median scaling
  repo: github.com__metabolomicstats__NormalizeMets
- name: statTarget
  role: Alternative/reference implementation of the QC-RLSC method as described by
    Dunn 2011
- name: R
  role: Runtime environment for the NormalizeMets functions; requires version 3.4.3
    or higher
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:e8bb1c1b4ddd4a6bab7b904caa393c706c9a498b1cd21f1cb1fc6eb3f47d94f7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NormalizeMets
  - R
  - statTarget
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# qc-sample-rlsc-signal-correction

## Summary

Corrects run-order drift in metabolomics feature data by applying the QC-RLSC (quality control sample based robust LOESS signal correction) method via NormalizeMets' NormQcsamples function, using QC samples analysed periodically throughout the batch.

## When to use

Use when metabolomics peak-intensity matrices (featuredata) show drift with run order and QC samples were injected periodically, so a QC-sample-based LOESS signal correction (rlsc) is needed before scaling or comparison.

## When NOT to use

- Normalization should be driven by internal/external standard metabolites or QC metabolites rather than QC samples — use NormQcmets (methods 'nomis', 'ccmn', 'is', 'ruv2') instead
- Only sample-wise median/mean/sum/reference scaling is needed with no run-order drift — use NormScaling rather than rlsc
- The dataset has no periodically injected QC samples, so QC-RLSC signal correction cannot be fit

## Inputs

- NormalizeMets featuredata matrix (samples as row names, metabolite peak intensities as columns)
- sampledata dataframe containing run order and sample type information (QC injections ordered by run order)
- Example dataset: Didata from the NormalizeMets package

## Outputs

- QC-RLSC corrected featuredata matrix (normalized metabolite abundances, samples in run order)
- CSV file of the corrected matrix when savefinaloutput=TRUE / saveoutput=TRUE (e.g. normcombined_results.csv)
- Optionally, a combined-normalization matrix when rlsc is chained with median scaling via NormCombined

## How to apply

Prepare the input in NormalizeMets' three-part format: a featuredata matrix (samples as rows, metabolites as columns), and a sampledata frame whose run order column is sorted before correction. Call NormQcsamples with method='rlsc' — the function implements the QC-RLSC method, which is also implemented in the statTarget package — using span=0, deg=2, lg=TRUE, ordering featuredata and sampledata by run order. For Didata, sort its featuredata and sampledata by the run order column, then pass both to NormQcsamples with method='rlsc'; rlsc can optionally be followed by median scaling via NormCombined(methods=c('rlsc','median')). Set savefinaloutput=TRUE with finaloutputname (e.g. 'normcombined_results') to write the corrected matrix as CSV. Verify the output CSV has metabolites as columns and samples as rows in the original run order, and that normalized values differ from raw unadjusted abundances, confirming the LOESS correction was applied.

## Related tools

- **NormalizeMets** (R package providing the NormQcsamples function that implements the QC-RLSC signal correction and NormCombined for chaining rlsc with median scaling) — github.com__metabolomicstats__NormalizeMets
- **statTarget** (Alternative/reference implementation of the QC-RLSC method as described by Dunn 2011)
- **R** (Runtime environment for the NormalizeMets functions; requires version 3.4.3 or higher) — https://cran.r-project.org/

## Examples

```
NormQcsamples(featuredata=Didata$featuredata, sampledata=Didata_sampledata_ord, method="rlsc", span=0, deg=2, lg=TRUE)
```

## Evaluation signals

- Corrected CSV exists, parses as valid CSV, and has metabolites as columns with samples as rows matching the input run-order ordering
- Normalized values differ from the raw unadjusted abundances, confirming the LOESS correction stage was actually applied
- NormQcsamples was called with the documented rlsc parameters span=0, deg=2, lg=TRUE on data sorted by run order
- QC injections used as anchors correspond to samples flagged as QC type in sampledata

## Limitations

- rlsc requires QC samples analysed periodically throughout the batch (run order information is mandatory); data without ordered QC injections cannot use this method
- The package handles data locally and reproduces results only if the exact sample ordering and rlsc tuning parameters (span, deg, lg) are preserved
- RLSC corrects signal drift only; residual unwanted variation typically still requires a second normalization step such as median scaling

## Evidence

- [readme] Workflow documentation entry point: [evidence span withheld — anchor status: external_doc]
