---
name: shiny-app-workflow-execution
description: Use when input is a table of compounds with aggregate statistics (p-value
  and fold-change) reported across multiple clinical studies, and the goal is combined
  per-compound meta-analysis via the Easy-Amanida Shiny app rather than raw-data re-analysis.
when_to_use_negative:
- Input data are raw individual-level measurements or spectra rather than aggregate
  statistics (p-value and fold-change) already reported per study — amanida operates
  on aggregate results
- You only need qualitative vote-counting without any identifier harmonization step
  and are not using the Easy-Amanida app workflow
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: Easy-Amanida
  role: Shiny web app that hosts the upload → harmonization → meta-analysis → export
    workflow for aggregate statistics
  repo: https://github.com/mariallr/easy-amanida
- name: amanida
  role: R package that computes the weighted combination of p-values and log2 fold-changes
    and the qualitative vote-counting
  repo: https://github.com/mariallr/easy-amanida
- name: webchem
  role: Back-end used to standardize heterogeneous compound identifiers (common names,
    InChIKeys, SMILES) onto a common chemical nomenclature
  repo: https://github.com/mariallr/easy-amanida
- name: Shiny
  role: R add-on framework in which the Easy-Amanida interactive web app is implemented
  repo: https://github.com/mariallr/easy-amanida
- name: R
  role: Environment in which the amanida meta-analysis is run and exported result
    tables (CSV) are validated
  repo: https://github.com/mariallr/easy-amanida
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_002
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:4cf07a22fda5abb1801760285c4f1e2630ab63c74d05cda3436d67a5dce31f5f
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# shiny-app-workflow-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Drive the Easy-Amanida Shiny app to run aggregate-statistics meta-analysis (combined p-value and fold-change) over clinical metabolomics compound lists, with webchem-backed naming harmonization. It packages the amanida R package behind an interactive upload-run-export workflow.

## When to use

Use when input is a table of compounds with aggregate statistics (p-value and fold-change) reported across multiple clinical studies, and the goal is combined per-compound meta-analysis via the Easy-Amanida Shiny app rather than raw-data re-analysis.

## When NOT to use

- Input data are raw individual-level measurements or spectra rather than aggregate statistics (p-value and fold-change) already reported per study — amanida operates on aggregate results
- You only need qualitative vote-counting without any identifier harmonization step and are not using the Easy-Amanida app workflow

## Inputs

- CSV/table of compound names with per-study aggregate statistics (p-value and fold-change), e.g. PDAC Supplementary Table 1 from 24 clinical studies
- Compound list with mixed identifier types (common names, InChIKeys, SMILES), e.g. colorectal-cancer Supplementary Table 3, when 'Include compounds IDs' is activated
- Choice of app option: 'Include compounds IDs' checked or unchecked

## Outputs

- amanida_metaanalysis_results.csv with combined p-value, log2 fold-change, and study count (N) per compound
- harmonization_results.csv reporting harmonized unique identifier count and per-compound vote counts
- vote_counting_results.csv summarizing qualitative vote-counting recurrence and trend direction per metabolite
- Counts of significant metabolites (p-value <0.05 and fold-change >2) and compounds appearing in more than one study

## How to apply

Load the per-study aggregate-statistics table (e.g. the PDAC Supplementary Table 1 with p-values and fold-changes from 24 clinical studies) and upload the compound list with statistics to the Easy-Amanida Shiny app at brui.shinyapps.io/easy-amanida, leaving 'Include compounds IDs' unchecked for common-name lists. Decide the identifier strategy: if the input mixes identifier types (common names, InChIKeys, SMILES, e.g. the colorectal-cancer Supplementary Table 3), activate 'Include compounds IDs' so the amanida/webchem back-end standardizes heterogeneous identifiers onto a common nomenclature. Run the amanida meta-analysis to compute the weighted combination of p-values and log2 fold-changes per compound (relying on webchem to unify synonyms), then export the combined per-compound results (e.g. amanida_metaanalysis_results.csv). Judge correctness by validating exported counts against reported reference values, such as 311 combined metabolites with 35 significant (p-value <0.05 and fold-change >2) for PDAC, or 133 IDs collapsing to 120 unique IDs with harmonization for the colorectal data.

## Related tools

- **Easy-Amanida** (Shiny web app that hosts the upload → harmonization → meta-analysis → export workflow for aggregate statistics) — https://github.com/mariallr/easy-amanida
- **amanida** (R package that computes the weighted combination of p-values and log2 fold-changes and the qualitative vote-counting) — https://github.com/mariallr/easy-amanida
- **webchem** (Back-end used to standardize heterogeneous compound identifiers (common names, InChIKeys, SMILES) onto a common chemical nomenclature) — https://github.com/mariallr/easy-amanida
- **Shiny** (R add-on framework in which the Easy-Amanida interactive web app is implemented) — https://github.com/mariallr/easy-amanida
- **R** (Environment in which the amanida meta-analysis is run and exported result tables (CSV) are validated) — https://github.com/mariallr/easy-amanida

## Evaluation signals

- PDAC quantitative run: 311 metabolites combined, 38 compounds appearing in more than one study, and 35 significant metabolites (p-value <0.05 and fold-change >2)
- Table 2 validation: Histidine combined p-value 0.023 with N 9500; Valine FC 160 up-regulated; Tryptophan FC 150 up-regulated
- Colorectal harmonization run: 133 mixed-type identifiers collapse to 120 unique IDs when 'Include compounds IDs' is activated
- With harmonization on, vote counts for hippuric acid and indole-3-acetic acid rise from 2 to 3
- Qualitative vote-counting: glutamine recurrent in 7 studies and histidine in 5 (both down-regulated), three bile acids (tauroursodeoxycholic, taurocholic, glycocholic acid) in 3 studies each, with 8 up / 12 down significant compounds after removing the 15 inconsistent-trend metabolites

## Limitations

- Leaving 'Include compounds IDs' unchecked with mixed identifier types (common name, InChIKey, SMILES) undercounts duplicates: the colorectal dataset yields 133 raw IDs instead of 120 unique IDs and vote counts of 2 instead of 3 for hippuric acid and indole-3-acetic acid
- Metabolites with inconsistent regulation trends across studies (15 flagged in the PDAC vote-counting) must be excluded from the significant-compound tally
- Compound naming synonyms in metabolomics studies require the webchem-backed harmonization step; results are otherwise not comparable across studies

## Evidence
