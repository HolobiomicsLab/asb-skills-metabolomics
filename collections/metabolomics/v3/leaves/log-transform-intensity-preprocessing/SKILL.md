---
name: log-transform-intensity-preprocessing
description: Use when metabolomics feature intensities are on a raw linear scale and
  are about to be fed into Metanorm's robust normalization methods, which expect log-transformed
  inputs.
when_to_use_negative:
- Input intensities are already on a log scale (e.g. log2 or log10) and re-transformation
  would distort the modeling assumptions
- The downstream method is SERFF, a multivariate normalization tool that is applied
  through its Shiny web interface rather than the Metanorm R pipeline and does not
  accept log-transformed inputs in the same way
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metanorm
  role: R package whose normalization functions (rLOESS, rGAM, tGAM, QC-RLSC, QC-RSC)
    consume the log-transformed intensity matrix; the log transform is the prerequisite
    preprocessing step before calling `metanorm()`.
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Language/runtime in which the Metanorm R package is executed; the log transform
    is applied via standard R operations (e.g. `log(rawdata)`) before normalization.
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:f70a4c7336aeb82bba44981338ccad6e143e4000ab3de07827f39834a5039cc0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm
  - R
  - SERFF
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# log-transform-intensity-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies natural-log (base e) transformation to raw metabolite intensities before downstream batchwise normalization (rLOESS, rGAM, tGAM, QC-RLSC, QC-RSC) so that the Metanorm models operate on a stabilized, multiplicative-noise-aware scale.

## When to use

Use when metabolomics feature intensities are on a raw linear scale and are about to be fed into Metanorm's robust normalization methods, which expect log-transformed inputs.

## When NOT to use

- Input intensities are already on a log scale (e.g. log2 or log10) and re-transformation would distort the modeling assumptions
- The downstream method is SERFF, a multivariate normalization tool that is applied through its Shiny web interface rather than the Metanorm R pipeline and does not accept log-transformed inputs in the same way

## Inputs

- Unnormalized numerical matrix of metabolite intensities (rows = compounds, columns = sample runs), e.g. from a FAME saliva feature table of 618 sample runs × 8913 metabolic features
- Batch vector assigning each sample run to a processing batch
- Sample-type vector (e.g. 'QC' vs 'sample') indicating which runs are quality-control samples

## Outputs

- Log-transformed (natural log, base e) intensity matrix ready for Metanorm normalization

## How to apply

Before calling the Metanorm R package's `metanorm()` function, take the unnormalized intensity matrix (rows = compounds, columns = sample runs, e.g. 618 FAME runs × 8913 metabolic features) and apply natural-log (base e) to every intensity value. The raw data matrix (`rawdata`), the batch vector, and the sample-type vector (`metanorm.qc` distinguishing 'QC' from other sample types) are loaded from the example RData or user-supplied equivalents; the log transform is applied to `rawdata` prior to batchwise normalization with rLOESS, rGAM, or tGAM (rGAM/tGAM with the QCcheck argument active). Rationale: the robust normalization models assume log-scaled signal where instrumental drift and batch effects behave as smooth additive shifts, and the example dataset in the package README shows log-scaled values (e.g. ~16.16–16.78 for the first compound).

## Related tools

- **Metanorm** (R package whose normalization functions (rLOESS, rGAM, tGAM, QC-RLSC, QC-RSC) consume the log-transformed intensity matrix; the log transform is the prerequisite preprocessing step before calling `metanorm()`.) — https://github.com/UGent-LIMET/Metanorm
- **R** (Language/runtime in which the Metanorm R package is executed; the log transform is applied via standard R operations (e.g. `log(rawdata)`) before normalization.) — https://cloud.r-project.org/index.html

## Examples

```
load(system.file("extdata", "example.RData", package = "metanorm")); logdata <- log(rawdata)
```

## Evaluation signals

- All intensity values in the output matrix are positive and bounded by the natural-log of the original max value; no zeros, negatives, or non-finite entries remain
- The first compound's logged values fall in the range observed in the package README example (≈16.16–16.78 for the first 20 sample runs)
- Downstream `metanorm()` call with model = "tGAM", type = metanorm.qc, QCcheck = TRUE, and batch = batch completes without errors and produces a `normdat` matrix of identical dimensions to the logged input
- PCA score plots generated with `plotPCA()` before and after normalization show diminished batch effects in the logged, normalized data

## Limitations

- The log transform alone does not address batch effects; it must be combined with Metanorm's batchwise robust normalization to be effective
- Natural-log (base e) is the specific convention used here — other log bases (log2, log10) are not what the article or README demonstrate and may change model behavior
- SERFF, a separate multivariate normalization method, failed to complete on the FAME dataset (618 × 8913) because it exceeded the server's capacity, so log transformation of that scale of data should not be expected to enable SERFF usage
- QC-RLSC and QC-RSC require the `QConly` argument to use QC samples exclusively; the log transform does not substitute for that decision

## Evidence

- [methods] Natural-log transform prior to normalization is the established preprocessing step for the FAME dataset before applying Metanorm's robust methods.: [evidence span withheld — anchor status: not_found]
- [readme] The README example shows the `rawdata` matrix already in a log-scale numeric range, confirming the expected input format.: [evidence span withheld — anchor status: external_doc]
