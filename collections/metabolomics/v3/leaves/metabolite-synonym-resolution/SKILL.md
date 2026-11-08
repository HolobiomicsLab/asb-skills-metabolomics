---
name: metabolite-synonym-resolution
description: Use when a metabolomics meta-analysis input table mixes identifier types
  (compound synonyms, InChIKeys, SMILES) so that the same compound appears under multiple
  labels and would otherwise inflate unique-ID counts and deflate vote counts.
when_to_use_negative:
- The compound list already uses a single consistent identifier scheme (e.g. all InChIKeys)
  with no synonym duplicates to resolve
- The input consists of raw spectra or individual-level (non-aggregate) data — Easy-Amanida/amanida
  is designed for aggregate statistics like p-value and fold-change
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2258
tools:
- name: Easy-Amanida
  role: Shiny web app providing the harmonization workflow and the 'Include compounds
    IDs' option
  repo: github.com/mariallr/easy-amanida
- name: amanida
  role: R package performing the aggregate-statistics meta-analysis on the harmonized
    compound list
  repo: github.com/mariallr/easy-amanida
- name: webchem
  role: R package used by the back-end to standardize heterogeneous compound identifiers
    to a common chemical nomenclature
  repo: github.com/mariallr/easy-amanida
- name: Shiny
  role: R add-on framework in which the Easy-Amanida interactive web app is implemented
- name: R
  role: Language in which amanida, webchem and the Easy-Amanida app are implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:300e07b9d7d8f82b7f96f4421ee0d8dce7b6b01de56c67978b2ff32b7da56e1b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolite-synonym-resolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes heterogeneous metabolite identifiers (common names, InChIKeys, SMILES) onto a common chemical nomenclature before aggregating meta-analysis statistics, as implemented in the Easy-Amanida Shiny app via the amanida and webchem R packages.

## When to use

Use when a metabolomics meta-analysis input table mixes identifier types (compound synonyms, InChIKeys, SMILES) so that the same compound appears under multiple labels and would otherwise inflate unique-ID counts and deflate vote counts.

## When NOT to use

- The compound list already uses a single consistent identifier scheme (e.g. all InChIKeys) with no synonym duplicates to resolve
- The input consists of raw spectra or individual-level (non-aggregate) data — Easy-Amanida/amanida is designed for aggregate statistics like p-value and fold-change

## Inputs

- CSV/table of compounds with aggregate statistics (p-value, fold-change) keyed by mixed identifier types: common names, InChIKeys, SMILES
- Colorectal cancer example dataset (Supplementary Table 3) with 133 compound identifiers

## Outputs

- harmonization_results.csv — harmonized compounds table mapping all input identifiers to a common ID (120 unique IDs from 133 inputs)
- Per-compound aggregated vote counts from the meta-analysis

## How to apply

Load the dataset (e.g. the colorectal cancer Supplementary Table 3 with 133 mixed-type compound identifiers) into the Easy-Amanida Shiny app. Activate the 'Include compounds IDs' harmonization option so the amanida and webchem back-end standardizes the heterogeneous identifiers onto a common chemical nomenclature; without it, only the given names are compared. Run the aggregate-statistics meta-analysis (p-value, fold-change) on the harmonized list and inspect the resulting unique identifier count and per-compound vote counts. Export the harmonized table as harmonization_results.csv. Validate against the reference outcome: 120 unique IDs (down from 133) and vote counts of 3 for both hippuric acid and indole-3-acetic acid (up from 2 without harmonization), demonstrating that synonyms of the same compound were merged and additional duplicates detected.

## Related tools

- **Easy-Amanida** (Shiny web app providing the harmonization workflow and the 'Include compounds IDs' option) — github.com/mariallr/easy-amanida
- **amanida** (R package performing the aggregate-statistics meta-analysis on the harmonized compound list) — github.com/mariallr/easy-amanida
- **webchem** (R package used by the back-end to standardize heterogeneous compound identifiers to a common chemical nomenclature) — github.com/mariallr/easy-amanida
- **Shiny** (R add-on framework in which the Easy-Amanida interactive web app is implemented)
- **R** (Language in which amanida, webchem and the Easy-Amanida app are implemented)

## Evaluation signals

- Unique identifier count drops from 133 to 120 after activating 'Include compounds IDs' on the colorectal cancer dataset
- Vote counts for hippuric acid and indole-3-acetic acid increase from 2 (unharmonized) to 3 (harmonized), confirming synonym-driven duplicates were merged
- harmonization_results.csv is exported and every input identifier (common name, InChIKey, or SMILES) resolves to a single common compound ID
- Aggregated p-values/fold-changes are computed per harmonized compound, not per raw identifier string
- No compound loses its aggregate statistics during harmonization (all 133 input rows are accounted for in the 120 harmonized entries)

## Limitations

- Harmonization depends on webchem being able to resolve the provided identifiers; unresolvable or ambiguous IDs may fail to map to the common nomenclature
- Without the 'Include compounds IDs' option, duplicates across identifier types are missed (e.g. vote counts stay at 2 instead of 3), so results depend on the option being enabled
- The article's discussion section content is absent from the provided context (no changelog found), so edge-case behavior of the harmonization is not documented here

## Evidence
