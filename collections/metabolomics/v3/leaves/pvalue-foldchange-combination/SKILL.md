---
name: pvalue-foldchange-combination
description: Use when you have aggregate statistics (p-value and fold-change) per
  compound from multiple clinical metabolomics studies rather than raw data, and need
  a combined significance call per metabolite with consistent naming across studies.
when_to_use_negative:
- Input is raw individual-level metabolomics data rather than pre-computed aggregate
  statistics (p-value and fold-change) — the amanida combination requires aggregate
  results
- Only a single study/dataset is being analyzed with no cross-study combination to
  perform (e.g. a single-dataset MetaboAnalyst-style analysis)
- Compound identifiers are already harmonized database IDs and 'Include compounds
  IDs' handling applies rather than name-synonym harmonization
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3391
tools:
- name: Easy-Amanida
  role: 'Web-based Shiny app that hosts the workflow: upload compound aggregate statistics,
    optimize naming combination, and run the meta-analysis'
  repo: https://github.com/mariallr/easy-amanida
- name: amanida
  role: R package that performs the weighted combination of p-values and log2 fold-changes
    per compound
  repo: https://github.com/mariallr/easy-amanida
- name: webchem
  role: R package used within Easy-Amanida to harmonize compound names/synonyms across
    contributing studies
  repo: https://github.com/mariallr/easy-amanida
- name: Shiny
  role: R package add-on implementing the Easy-Amanida interactive web app
  repo: https://github.com/mariallr/easy-amanida
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:4807fee1390dc81627a376675bf68ee9eaece71676aa8f5bfc64ca1bbfa3c2a1
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# pvalue-foldchange-combination

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantitative meta-analysis of aggregate metabolomics statistics by combining p-values and log2 fold-changes across studies with the amanida R package, while harmonizing compound name synonyms via webchem inside the Easy-Amanida Shiny app.

## When to use

Use when you have aggregate statistics (p-value and fold-change) per compound from multiple clinical metabolomics studies rather than raw data, and need a combined significance call per metabolite with consistent naming across studies.

## When NOT to use

- Input is raw individual-level metabolomics data rather than pre-computed aggregate statistics (p-value and fold-change) — the amanida combination requires aggregate results
- Only a single study/dataset is being analyzed with no cross-study combination to perform (e.g. a single-dataset MetaboAnalyst-style analysis)
- Compound identifiers are already harmonized database IDs and 'Include compounds IDs' handling applies rather than name-synonym harmonization

## Inputs

- Supplementary table of aggregate statistics (p-value and fold-change per compound per study), e.g. PDAC Supplementary Table 1 from 24 clinical studies
- PR000293 lung cancer blood dataset from Metabolomics Workbench (Supplementary Table 2) with aggregate statistics per study
- Compound name lists with synonyms across contributing studies

## Outputs

- Combined per-compound results table (e.g. amanida_metaanalysis_results.csv) with combined p-values and fold-changes
- List of significant metabolites meeting p-value <0.05 and fold-change >2
- Flagged compounds with inconsistent fold-change trends (e.g. salicylic acid in PR000293)

## How to apply

Load the supplementary table of aggregate statistics (p-value, fold-change, study) into R and upload the compound list to the Easy-Amanida Shiny app (leave 'Include compounds IDs' unchecked when IDs are absent). Use the webchem-backed naming harmonization workflow to unify compound synonyms across contributing studies before combination. Run the amanida meta-analysis to compute the weighted combination of p-values and log2 fold-changes per compound. Flag metabolites as significant when combined p-value < 0.05 and fold-change > 2 across multiple studies, and export per-compound results (e.g. amanida_metaanalysis_results.csv). The rationale is that metabolomics meta-analysis is hindered by compound naming synonyms and inconsistent statistical reporting, so combination must happen on harmonized identifiers of aggregate statistics.

## Related tools

- **Easy-Amanida** (Web-based Shiny app that hosts the workflow: upload compound aggregate statistics, optimize naming combination, and run the meta-analysis) — https://github.com/mariallr/easy-amanida
- **amanida** (R package that performs the weighted combination of p-values and log2 fold-changes per compound) — https://github.com/mariallr/easy-amanida
- **webchem** (R package used within Easy-Amanida to harmonize compound names/synonyms across contributing studies) — https://github.com/mariallr/easy-amanida
- **Shiny** (R package add-on implementing the Easy-Amanida interactive web app) — https://github.com/mariallr/easy-amanida

## Evaluation signals

- On the PDAC benchmark (24 studies), the run combines 311 metabolites, 38 compounds appear in more than one study, and 35 metabolites are significant at p<0.05 and FC>2
- Table 2 anchor values reproduce: Histidine combined p-value 0.023 with N 9500; Valine FC 160 up-regulated; Tryptophan FC 150 up-regulated
- On PR000293 lung cancer blood data, three metabolites are consistently significant in both the amanida combination and the MetaboAnalyst single-dataset analysis, with salicylic acid significant in the meta-analysis but showing inconsistent trends
- Exported per-compound results file (amanida_metaanalysis_results.csv) contains one combined p-value and fold-change per harmonized compound
- Compound count before and after naming harmonization confirms synonyms were unified (e.g. distinct raw names collapse onto harmonized identifiers)

## Limitations

- Requires aggregate statistics (p-value, fold-change); meta-analyses in metabolomics are hindered by lack of consensus on statistical reporting, so some studies may lack the needed metrics
- Compound naming synonyms across studies must be resolved by the webchem harmonization workflow; failures there propagate into the combined results
- Fold-change direction can be inconsistent across studies (e.g. salicylic acid flagged with inconsistent trends), which the combination does not silently resolve

## Evidence
