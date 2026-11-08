---
name: aggregate-p-value-combination
description: Use when contributing studies report only aggregate statistics (p-value,
  fold-change, sample size per compound) and you need a combined meta-analysis estimate
  per compound without access to individual-level data.
when_to_use_negative:
- Studies provide individual-level (raw) data rather than aggregate statistics — a
  standard individual-data meta-analysis approach applies instead
- Per-study tables lack p-values and fold-changes (e.g. only qualitative compound
  lists), since Amanida's combination requires both aggregate metrics
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: amanida
  role: R package performing the weighted Fisher's-method p-value combination and
    weighted log2 fold-change averaging on aggregate statistics
  repo: github.com/mariallr/easy-amanida
- name: Easy-Amanida
  role: Shiny web app wrapping amanida and webchem, providing a workflow to optimize
    compound naming combination before meta-analysis
  repo: github.com/mariallr/easy-amanida
- name: webchem
  role: Supports compounds naming harmonization when combining aggregate data in Easy-Amanida
- name: Shiny
  role: R package add-on in which the Easy-Amanida web app for this combination workflow
    is implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:ebef52a15a6a43379734d9bb31295b37e19b9250019fc64eda961490fa7801fc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/easy-amanida
  tools:
  - Easy-Amanida
  - R
  - Shiny
  - amanida
  - webchem
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# aggregate-p-value-combination

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines per-study p-values and fold-changes reported only as aggregate statistics using Amanida's weighted Fisher's method (gamma-distribution, study-size-proportional weights) plus weighted averaging of log2-transformed fold-changes, as implemented in the R package amanida and the Easy-Amanida Shiny app.

## When to use

Use when contributing studies report only aggregate statistics (p-value, fold-change, sample size per compound) and you need a combined meta-analysis estimate per compound without access to individual-level data.

## When NOT to use

- Studies provide individual-level (raw) data rather than aggregate statistics — a standard individual-data meta-analysis approach applies instead
- Per-study tables lack p-values and fold-changes (e.g. only qualitative compound lists), since Amanida's combination requires both aggregate metrics

## Inputs

- Per-study aggregate statistics table (CSV) with columns: compound identifier, p-value, fold-change, sample size per study
- Compound identifier lists requiring naming harmonization across studies

## Outputs

- amanida_meta_analysis_results.csv with combined p-value, combined log2 fold-change, and regulation direction (up/down) per compound
- Naming-harmonized compound table grouping rows of the same compound across studies

## How to apply

Load the per-study aggregate statistics table (compound identifier, p-value, fold-change, sample size per study), e.g. from the demo dataset in the mariallr/easy-amanida GitHub repository, and harmonize compound identifiers so rows referring to the same compound are grouped — Amanida's naming-combination workflow, which can be optimized via webchem-backed harmonization in Easy-Amanida. For each compound, combine the per-study p-values with Amanida's weighted Fisher's method, in which a gamma distribution assigns non-integral weights to each p-value proportional to the study's sample size. In parallel, compute the weighted mean of log2-transformed (base 2) fold-changes with weights proportional to study sample size. Classify each compound as up- or down-regulated from the sign of the combined log2 fold-change, and write the combined p-value, combined fold-change, and regulation direction per compound to amanida_meta_analysis_results.csv.

## Related tools

- **amanida** (R package performing the weighted Fisher's-method p-value combination and weighted log2 fold-change averaging on aggregate statistics) — github.com/mariallr/easy-amanida
- **Easy-Amanida** (Shiny web app wrapping amanida and webchem, providing a workflow to optimize compound naming combination before meta-analysis) — github.com/mariallr/easy-amanida
- **webchem** (Supports compounds naming harmonization when combining aggregate data in Easy-Amanida)
- **Shiny** (R package add-on in which the Easy-Amanida web app for this combination workflow is implemented)

## Examples

```
Rscript -e 'library(amanida); res <- amanida(data, type = "meta"); write.csv(res, "amanida_meta_analysis_results.csv")'
```

## Evaluation signals

- Each compound's combined p-value matches Amanida's weighted Fisher's method output using gamma-derived, sample-size-proportional non-integral weights
- Combined fold-change equals the sample-size-weighted mean of base-2 log-transformed per-study fold-changes
- Regulation direction per compound agrees with the sign of the combined log2 fold-change
- Output CSV amanida_meta_analysis_results.csv contains one row per compound with combined p-value, combined fold-change, and direction columns
- Rows referring to the same compound across studies are grouped under a single harmonized identifier before combination

## Limitations

- Requires aggregate statistics (p-value, fold-change, sample size) per study; metabolomics meta-analysis is hindered by lack of consensus on statistical reporting
- Compound naming synonyms across studies must be harmonized before combination, and naming combination may need optimization via the Easy-Amanida workflow

## Evidence

- [methods] weighted Fisher's method with gamma weights: [evidence span withheld — anchor status: not_found]
- [abstract] aggregate statistics meta-analysis with naming harmonization: "combines two R packages, "amanida" and "webchem", to enable meta-analysis of aggregate statistical data, like p-value and fold-change, while ensuring the compounds naming harmonization"
- [intro] Shiny implementation: "Easy-Amanida is a web-based app, developed with Shiny, that implements the R package Amanida."
- [methods] naming-combination optimization workflow: "provides a workflow to optimize the naming combination"
- [readme] app description: "The Easy-Amanida app is implemented in Shiny, an R package add-on for interactive web apps, and provides a workflow to optimize naming combination."
