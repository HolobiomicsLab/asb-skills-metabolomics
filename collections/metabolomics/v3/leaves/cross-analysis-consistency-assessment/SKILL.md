---
name: cross-analysis-consistency-assessment
description: Use when you have aggregate statistics (p-values, fold-changes) per study
  and need to verify that a meta-analysis tool's combined significance calls agree
  with an independent single-dataset analysis of the same data.
when_to_use_negative:
- Input is raw spectral data or a single-study feature table rather than aggregate
  statistics (p-value, fold-change) reported per study
- Compound naming harmonization is not needed and per-study identifiers are already
  consistent across studies
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Easy-Amanida
  role: Shiny web app that implements the meta-analysis workflow over aggregate statistics
    with compound name harmonization
  repo: https://github.com/mariallr/easy-amanida
- name: amanida
  role: R package combining p-values and fold-changes into a quantitative meta-analysis
    of aggregate data
  repo: https://github.com/mariallr/easy-amanida
- name: webchem
  role: R package used within Easy-Amanida to harmonize compound names across contributing
    studies
  repo: https://github.com/mariallr/easy-amanida
- name: Shiny
  role: R add-on package implementing the Easy-Amanida interactive web app
  repo: https://github.com/mariallr/easy-amanida
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:ad7736e262f2b7762519294cab49eab79b3c848d6a7ee75ac226091b6f9c863f
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cross-analysis-consistency-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assesses whether a quantitative meta-analysis performed with Easy-Amanida (amanida + webchem) yields significance calls consistent with an independent analysis (e.g., MetaboAnalyst) of the same dataset, by comparing metabolite-level significance across both analyses.

## When to use

Use when you have aggregate statistics (p-values, fold-changes) per study and need to verify that a meta-analysis tool's combined significance calls agree with an independent single-dataset analysis of the same data.

## When NOT to use

- Input is raw spectral data or a single-study feature table rather than aggregate statistics (p-value, fold-change) reported per study
- Compound naming harmonization is not needed and per-study identifiers are already consistent across studies

## Inputs

- Aggregate statistics table per study (p-value, fold-change) for a metabolomics dataset, e.g., Metabolomics Workbench PR000293 Supplementary Table 2
- Preprocessed matrix with log-transformed, autoscaled metabolite features
- Compound name list with synonyms across contributing studies

## Outputs

- Combined quantitative meta-analysis result per metabolite (combined p-value and fold-change from amanida)
- Harmonized compound naming table from webchem
- Consistency call list: metabolites significant in both analyses, plus discordant calls (e.g., salicylic acid with inconsistent trends)

## How to apply

Load the aggregate-statistics table (e.g., PR000293 lung cancer blood metabolomics data from Metabolomics Workbench, Supplementary Table 2) and preprocess it by log-transforming and autoscaling as done in MetaboAnalyst. Harmonize compound names across contributing studies using the webchem R package inside Easy-Amanida, since metabolomics compound synonyms otherwise block name combination. Run the amanida R package to combine p-values and fold-changes into a quantitative meta-analysis. Then compare the combined significance calls against the independent analysis to identify metabolites significant in both — e.g., three metabolites consistently significant, with salicylic acid significant in Easy-Amanida but showing inconsistent trends. Judge success by agreement of the metabolite-level significance overlap with the reported result (a qualitative acceptance check).

## Related tools

- **Easy-Amanida** (Shiny web app that implements the meta-analysis workflow over aggregate statistics with compound name harmonization) — https://github.com/mariallr/easy-amanida
- **amanida** (R package combining p-values and fold-changes into a quantitative meta-analysis of aggregate data) — https://github.com/mariallr/easy-amanida
- **webchem** (R package used within Easy-Amanida to harmonize compound names across contributing studies) — https://github.com/mariallr/easy-amanida
- **Shiny** (R add-on package implementing the Easy-Amanida interactive web app) — https://github.com/mariallr/easy-amanida

## Examples

```
Rscript -e "library(amanida); library(webchem); res <- amanida::compute_amanida('PR000293_aggregate.csv')"
```

## Evaluation signals

- Metabolites significant in both Easy-Amanida and the MetaboAnalyst analysis are enumerated and count matches the reported overlap (three metabolites)
- Discordant calls are flagged with trend direction: salicylic acid significant in Easy-Amanida meta-analysis but with inconsistent fold-change trends
- Compound naming harmonization step (webchem) completes without unmapped synonyms before amanida combination is run
- Aggregate statistics were preprocessed by log-transformation and autoscaling matching the MetaboAnalyst procedure before comparison
- Qualitative acceptance check: output matches the reported finding of three consistently significant metabolites plus salicylic acid

## Limitations

- Meta-analysis in metabolomics is hindered by wide compound variety, differing experimental methods, and lack of consensus on statistical reporting
- Compound naming synonyms across studies must be harmonized or combination results are unreliable
- Validation is qualitative (matching the reported finding), not a formal statistical test of consistency

## Evidence
