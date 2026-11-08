---
name: metabolomics-dataset-preprocessing
description: Use when combining metabolomics results from multiple studies reported
  only as aggregate statistics (p-value, fold-change), where compound name synonyms
  across studies must be resolved before meta-analysis.
when_to_use_negative:
- Input is raw spectral data (e.g. mzML/NetCDF) rather than already-computed aggregate
  statistics (p-value, fold-change) — Easy-Amanida operates on aggregate data only
- Only a single dataset is available, so no cross-study naming harmonization or meta-analysis
  is needed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Easy-Amanida
  role: Web-based Shiny app that implements the preprocessing-to-meta-analysis workflow,
    combining amanida and webchem for aggregate data with compound naming harmonization
  repo: https://github.com/mariallr/easy-amanida
- name: amanida
  role: R package that combines p-values and fold-changes into a quantitative meta-analysis
    of aggregate data
  repo: https://github.com/mariallr/easy-amanida
- name: webchem
  role: R package used within Easy-Amanida to harmonize compound names across contributing
    studies
- name: Shiny
  role: R package add-on implementing the interactive web app that drives the workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:e35d0ca26caeaf6f71a650c2e31fa6fe2590105b82db225e3bba17b8fcedcb1b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/easy-amanida
  repo_url: https://github.com/mariallr/easy-amanida
  tools:
  - Easy-Amanida
  - R
  - Shiny
  - amanida
  - webchem
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# metabolomics-dataset-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess metabolomics aggregate statistics (p-values and fold-changes) and harmonize compound naming across studies so that quantitative meta-analysis (e.g., with amanida/Easy-Amanida) can be performed consistently with tools like MetaboAnalyst.

## When to use

Use when combining metabolomics results from multiple studies reported only as aggregate statistics (p-value, fold-change), where compound name synonyms across studies must be resolved before meta-analysis.

## When NOT to use

- Input is raw spectral data (e.g. mzML/NetCDF) rather than already-computed aggregate statistics (p-value, fold-change) — Easy-Amanida operates on aggregate data only
- Only a single dataset is available, so no cross-study naming harmonization or meta-analysis is needed

## Inputs

- Aggregate statistics table per study (p-value and fold-change), e.g. Metabolomics Workbench PR000293 Supplementary Table 2
- Compound name lists from contributing studies (with synonyms)
- Raw metabolite concentrations or fold-changes requiring log-transformation and autoscaling

## Outputs

- Log-transformed, autoscaled aggregate statistics table
- Harmonized compound naming mapping across studies
- Combined meta-analysis results (p-values and fold-changes per metabolite)

## How to apply

Load the aggregate-statistics table (e.g., Metabolomics Workbench dataset PR000293 Supplementary Table 2) containing per-study p-values and fold-changes. Preprocess the statistics by log-transforming and autoscaling, mirroring the MetaboAnalyst workflow, so combined results are comparable to a single-dataset analysis. Harmonize compound naming across contributing studies using the webchem R package, which Easy-Amanida wraps to provide a workflow that optimizes the naming combination despite metabolomics synonyms and differing experimental methods. Feed the harmonized, preprocessed table into the amanida R package (via the Shiny-based Easy-Amanida app) to combine p-values and fold-changes into a quantitative meta-analysis, then compare combined significance calls against single-dataset calls to identify consistently significant metabolites.

## Related tools

- **Easy-Amanida** (Web-based Shiny app that implements the preprocessing-to-meta-analysis workflow, combining amanida and webchem for aggregate data with compound naming harmonization) — https://github.com/mariallr/easy-amanida
- **amanida** (R package that combines p-values and fold-changes into a quantitative meta-analysis of aggregate data) — https://github.com/mariallr/easy-amanida
- **webchem** (R package used within Easy-Amanida to harmonize compound names across contributing studies)
- **Shiny** (R package add-on implementing the interactive web app that drives the workflow)

## Examples

```
shiny::runGitHub("mariallr/easy-amanida")  # or use https://brui.shinyapps.io/easy-amanida
```

## Evaluation signals

- After log-transformation and autoscaling, combined significance calls reproduce the MetaboAnalyst single-dataset analysis (e.g., three metabolites consistently significant for PR000293)
- All compound synonyms across contributing studies are resolved to a single harmonized identifier before meta-analysis
- Salicylic acid is flagged as significant in the Easy-Amanida meta-analysis but with inconsistent trends, matching the reported result
- The combined output contains both a p-value and a fold-change for every metabolite entry submitted
- The Easy-Amanida naming-combination workflow reports an optimized naming mapping rather than leaving unresolved synonyms

## Limitations

- Metabolomics meta-analysis is hindered by wide compound variety, differing experimental methods, lack of consensus on statistical reporting, and compound naming synonyms
- Requires aggregate statistics (p-value, fold-change) as input; it cannot operate on raw spectra or individual-level data
- Trend inconsistency can occur (e.g., salicylic acid significant in Easy-Amanida but showing inconsistent trends), requiring qualitative check of fold-change direction

## Evidence

- [other] PR000293 workflow: [evidence span withheld — anchor status: not_found]
