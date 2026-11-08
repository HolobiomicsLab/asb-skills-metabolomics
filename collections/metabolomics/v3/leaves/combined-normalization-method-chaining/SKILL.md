---
name: combined-normalization-method-chaining
description: Use when a metabolomics featuredata matrix needs multiple normalization
  layers, e.g. rlsc signal correction of QC-sample drift followed by median scaling,
  in the NormalizeMets R workflow.
when_to_use_negative:
- Only one normalization layer is needed — use NormQcmets, NormQcsamples or NormScaling
  directly
- The dataset lacks QC sample run-order information needed by the rlsc signal-correction
  stage
- Data is not yet in NormalizeMets featuredata/sampledata format (unique sample names
  as row names, metabolites as column names)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NormalizeMets
  role: R package providing the NormCombined function that chains the rlsc and median
    normalization methods
  repo: https://cran.r-project.org/
- name: statTarget
  role: Implements the QC-RLSC (robust LOESS) signal correction underlying the rlsc
    method invoked first in the chain
- name: R
  role: Runtime environment; NormalizeMets requires R version 3.4.3 or higher
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:fb9a4f7ac1fb58adaa3955c0d56df02237aaaabe71adffd0b472479982fa2b1d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NormalizeMets
  - R
  - statTarget
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# combined-normalization-method-chaining

## Summary

Chains two normalization methods in sequence using the NormalizeMets package's NormCombined function — by default QC-RLSC signal correction (rlsc) followed by median scaling — for metabolomics featuredata matrices where a single method is insufficient.

## When to use

Use when a metabolomics featuredata matrix needs multiple normalization layers, e.g. rlsc signal correction of QC-sample drift followed by median scaling, in the NormalizeMets R workflow.

## When NOT to use

- Only one normalization layer is needed — use NormQcmets, NormQcsamples or NormScaling directly
- The dataset lacks QC sample run-order information needed by the rlsc signal-correction stage
- Data is not yet in NormalizeMets featuredata/sampledata format (unique sample names as row names, metabolites as column names)

## Inputs

- featuredata matrix (metabolite peak intensities/concentrations, samples as rows, metabolites as columns)
- sampledata dataframe with sample-specific information including run order
- optional methods vector, default c('rlsc','median')

## Outputs

- combined normalized featuredata matrix (rlsc-corrected then median-scaled)
- optional CSV file normcombined_results.csv when savefinaloutput=TRUE

## How to apply

Load a metabolomics dataset in NormalizeMets format (e.g. the Didata example) and order featuredata and sampledata by the run-order column in sampledata. Call NormCombined(featuredata, sampledata, methods = c('rlsc','median')) leaving the default method order: the rlsc method (QC-sample based robust LOESS signal correction, as implemented in statTarget per Luan et al. 2017) is applied first to correct unwanted variation, then the median scaling normalization (NormScaling with method 'median') is chained on the rlsc-corrected matrix. Set savefinaloutput=TRUE with finaloutputname='normcombined_results' so the combined normalized featuredata matrix is written to normcombined_results.csv. Validate that the output CSV has metabolites as columns and samples as rows in the input sample order and that values differ from raw unadjusted abundances, confirming both stages ran. The function also accepts additional arguments passed via '...' to the underlying NormQcsamples and NormScaling calls.

## Related tools

- **NormalizeMets** (R package providing the NormCombined function that chains the rlsc and median normalization methods) — https://cran.r-project.org/
- **statTarget** (Implements the QC-RLSC (robust LOESS) signal correction underlying the rlsc method invoked first in the chain)
- **R** (Runtime environment; NormalizeMets requires R version 3.4.3 or higher) — https://cran.r-project.org/

## Examples

```
R: library(NormalizeMets); res <- NormCombined(featuredata, sampledata, methods=c('rlsc','median'), savefinaloutput=TRUE, finaloutputname='normcombined_results')
```

## Evaluation signals

- normcombined_results.csv exists and is a valid CSV with metabolites as columns and samples as rows
- Output sample ordering matches the input sampledata run order
- Normalized values differ from raw unadjusted abundances, indicating both rlsc and median stages applied
- The default methods vector is c('rlsc','median') — order matters; rlsc must precede median
- Rows/columns match the NormalizeMets featuredata format (unique sample names as row names, unique metabolite names as column names)

## Limitations

- The combined chain is fixed by the 'methods' argument order; swapping e.g. c('median','rlsc') would apply steps in a different order than the documented default
- rlsc relies on quality control samples and run order, so datasets without QC samples injected at regular intervals cannot use the default chain
- Requires R (≥3.4.3) and the NormalizeMets package; Excel users need Microsoft Excel 2016+ on Windows 7 or higher

## Evidence

- [methods] Combined normalization step (NormCombined): "researchers use a combination of the above normalizations (i.e., one method followed by another). This can be achieved using the _NormCombined_ function."
- [methods] NormCombined signature: "NormCombined<-function(featuredata, methods = c("rlsc", "median"), savefinaloutput = FALSE, finaloutputname = NULL, ...)"
- [methods] rlsc underlying method: "quality control sample based robust LOESS (locally estimated scatterplot smoothing) signal correction (QC-RLSC) method as described by @Dunn2011 and impletemented statTarget [@Luan2017]"
- [readme] README package purpose: [evidence span withheld — anchor status: external_doc]
- [readme] README input format: [evidence span withheld — anchor status: external_doc]
