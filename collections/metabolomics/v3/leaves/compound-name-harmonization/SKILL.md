---
name: compound-name-harmonization
description: Use when a metabolomics meta-analysis input table contains mixed identifier
  types or compound synonyms across contributing studies, causing the same compound
  to appear as multiple rows with split vote counts.
when_to_use_negative:
- The input identifier list already uses a single, consistent identifier type with
  no synonyms — harmonization would be a no-op.
- You have raw individual-level data rather than disclosed aggregate statistics (p-value,
  fold-change); the amanida pipeline is designed for aggregate-statistics meta-analysis.
- The task does not require cross-study grouping of compounds (e.g. a single-study
  analysis where naming harmonization adds no value).
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Easy-Amanida
  role: Shiny web app exposing the 'Include compounds IDs' harmonization option and
    the naming-combination workflow
  repo: https://github.com/mariallr/easy-amanida
- name: webchem
  role: R package back-end that standardizes heterogeneous compound identifiers to
    a common chemical nomenclature
  repo: https://github.com/mariallr/easy-amanida
- name: amanida
  role: Performs the naming combination that groups rows referring to the same compound
    before combining p-values and fold-changes
  repo: https://github.com/mariallr/easy-amanida
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:2005af91dcead3510a9e84cbb94b1688592a814167823880e0234f13a8709334
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
  - amanida
  - webchem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# compound-name-harmonization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes heterogeneous compound identifiers (common names, synonyms, InChIKeys, SMILES) into a common chemical nomenclature before aggregate-statistics meta-analysis, using the webchem-backed naming combination workflow of the Easy-Amanida Shiny app.

## When to use

Use when a metabolomics meta-analysis input table contains mixed identifier types or compound synonyms across contributing studies, causing the same compound to appear as multiple rows with split vote counts.

## When NOT to use

- The input identifier list already uses a single, consistent identifier type with no synonyms — harmonization would be a no-op.
- You have raw individual-level data rather than disclosed aggregate statistics (p-value, fold-change); the amanida pipeline is designed for aggregate-statistics meta-analysis.
- The task does not require cross-study grouping of compounds (e.g. a single-study analysis where naming harmonization adds no value).

## Inputs

- Compound list with aggregate statistics per study (p-value, fold-change, sample size) as in PDAC Supplementary Table 1 or PR000293 Supplementary Table 2
- Mixed-type compound identifiers: common names, InChIKeys, and SMILES (e.g. colorectal cancer Supplementary Table 3 with 133 identifiers)

## Outputs

- Harmonized compound table with identifiers standardized to a common chemical nomenclature and rows grouped per compound
- Per-compound vote counts reflecting merged duplicates (e.g. hippuric acid and indole-3-acetic acid with 3 votes instead of 2)
- Exported results file such as harmonization_results.csv

## How to apply

Load the per-study aggregate-statistics table (compound identifier, p-value, fold-change, sample size) into the Easy-Amanida Shiny app. Activate the 'Include compounds IDs' option so the amanida and webchem back-end standardizes the heterogeneous identifiers (common names, InChIKeys, SMILES) onto a common chemical nomenclature, unifying compound synonyms. Then run the naming-combination workflow so that rows referring to the same compound are grouped before computing the weighted Fisher combination of p-values and the sample-size-weighted mean of log2 fold-changes. Rationale: the wide variety of compounds and naming synonyms across studies hinder metabolomics meta-analysis, so harmonization must precede combination or duplicates will be missed and votes split. Export the harmonized per-compound table (e.g. harmonization_results.csv) and check that unique identifier counts decreased and vote counts increased relative to the unharmonized run.

## Related tools

- **Easy-Amanida** (Shiny web app exposing the 'Include compounds IDs' harmonization option and the naming-combination workflow) — https://github.com/mariallr/easy-amanida
- **webchem** (R package back-end that standardizes heterogeneous compound identifiers to a common chemical nomenclature) — https://github.com/mariallr/easy-amanida
- **amanida** (Performs the naming combination that groups rows referring to the same compound before combining p-values and fold-changes) — https://github.com/mariallr/easy-amanida

## Evaluation signals

- Unique identifier count decreases after harmonization (133 mixed IDs reduced to 120 unique IDs in the colorectal cancer validation).
- Per-compound vote counts increase versus the unharmonized run (hippuric acid: 3 vs 2; indole-3-acetic acid: 3 vs 2).
- Additional duplicates are identified after standardization that were missed without the 'Include compounds IDs' option.
- Compound synonyms across the 24 PDAC studies are unified (e.g. 311 metabolites combined, 38 compounds appearing in more than one study).
- The exported harmonization_results.csv rows group all study entries of the same compound under a single harmonized identifier.

## Limitations

- Harmonization relies on the webchem-backed identifier resolution; identifiers it cannot map to a common nomenclature may remain unmerged.
- The 'Include compounds IDs' option changes results (fewer unique IDs, higher vote counts), so harmonized and unharmonized runs are not directly comparable without noting the option state.
- Discussion-section documentation is absent from the retrieved article content, so edge-case behavior (e.g. failed identifier lookups) is not characterized.

## Evidence
