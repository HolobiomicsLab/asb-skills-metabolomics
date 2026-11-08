---
name: mixed-identifier-type-mapping
description: Use when a metabolomics meta-analysis input table mixes identifier types
  (common name, InChIKey, SMILES) for the same compounds, inflating unique-ID counts
  and masking duplicate votes across studies.
when_to_use_negative:
- The input compound list already uses a single consistent identifier type (e.g. all
  InChIKeys), so harmonization adds no resolution of synonyms
- The analysis goal is raw spectral processing or feature detection rather than meta-analysis
  of already-aggregated statistics (p-value, fold-change)
- Compounds lack resolvable machine-readable IDs (InChIKey/SMILES) that webchem can
  map to common nomenclature
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Easy-Amanida
  role: Shiny web app exposing the 'Include compounds IDs' harmonization option and
    the meta-analysis workflow
  repo: https://github.com/mariallr/easy-amanida
- name: amanida
  role: R package back-end performing aggregate-statistics meta-analysis and vote
    counting on the harmonized compound list
  repo: https://github.com/mariallr/easy-amanida
- name: webchem
  role: R package back-end that standardizes mixed identifiers (common name, InChIKey,
    SMILES) onto a common chemical nomenclature
  repo: https://github.com/mariallr/easy-amanida
- name: Shiny
  role: Interactive web-app framework in which Easy-Amanida is implemented
- name: R
  role: Language in which amanida, webchem and the Easy-Amanida app are implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:95d0bf2086db5dc73404341ad9c236d0e82da094c97b860283729c87c993ae19
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# mixed-identifier-type-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes heterogeneous compound identifiers (common names, InChIKeys, SMILES) onto a single common chemical nomenclature using Easy-Amanida's 'Include compounds IDs' option, which drives the amanida and webchem back-end, before running aggregate-statistics meta-analysis.

## When to use

Use when a metabolomics meta-analysis input table mixes identifier types (common name, InChIKey, SMILES) for the same compounds, inflating unique-ID counts and masking duplicate votes across studies.

## When NOT to use

- The input compound list already uses a single consistent identifier type (e.g. all InChIKeys), so harmonization adds no resolution of synonyms
- The analysis goal is raw spectral processing or feature detection rather than meta-analysis of already-aggregated statistics (p-value, fold-change)
- Compounds lack resolvable machine-readable IDs (InChIKey/SMILES) that webchem can map to common nomenclature

## Inputs

- Compound table with mixed identifier types (common names, InChIKeys, SMILES) plus aggregate statistics (p-value, fold-change), e.g. Supplementary Table 3 with 133 identifiers
- Easy-Amanida Shiny app session with the 'Include compounds IDs' option toggleable

## Outputs

- Harmonized compound list with identifiers standardized to a common chemical nomenclature
- harmonization_results.csv with unique identifier counts (120 unique IDs from 133) and per-compound vote counts
- Aggregate meta-analysis results (combined p-values and fold-changes) with updated duplicate detection

## How to apply

Load the compound table with its mixed-type identifiers (e.g. the colorectal cancer example dataset, Supplementary Table 3, holding 133 identifiers of common-name, InChIKey and SMILES types) into the Easy-Amanida Shiny app. Activate the 'Include compounds IDs' harmonization option so the amanida and webchem back-end standardizes the heterogeneous identifiers onto one common chemical nomenclature; without this option, synonyms of the same metabolite are counted as distinct. Re-run the aggregate-statistics meta-analysis (combining p-values and fold-changes) on the harmonized list. Export the harmonized results as harmonization_results.csv and compare against the non-harmonized run to quantify how many apparent unique identifiers were actually duplicates of the same compound.

## Related tools

- **Easy-Amanida** (Shiny web app exposing the 'Include compounds IDs' harmonization option and the meta-analysis workflow) — https://github.com/mariallr/easy-amanida
- **amanida** (R package back-end performing aggregate-statistics meta-analysis and vote counting on the harmonized compound list) — https://github.com/mariallr/easy-amanida
- **webchem** (R package back-end that standardizes mixed identifiers (common name, InChIKey, SMILES) onto a common chemical nomenclature) — https://github.com/mariallr/easy-amanida
- **Shiny** (Interactive web-app framework in which Easy-Amanida is implemented)
- **R** (Language in which amanida, webchem and the Easy-Amanida app are implemented)

## Evaluation signals

- Unique identifier count drops from 133 to 120 after activating 'Include compounds IDs' on the colorectal cancer dataset (Supplementary Table 3)
- Vote counts for hippuric acid and indole-3-acetic acid increase from 2 to 3, confirming previously masked duplicates were merged
- harmonization_results.csv exports successfully and contains the harmonized common nomenclature column
- Every compound in the output is represented by one canonical identifier despite mixed input types (common name, InChIKey, SMILES)

## Limitations

- Without activating 'Include compounds IDs', synonyms across identifier types remain as separate entries, undercounting duplicates and votes
- Harmonization depends on webchem being able to resolve each identifier; unresolvable or malformed identifiers may not map to common nomenclature
- Tool is designed for aggregate statistical data (p-value, fold-change), not raw metabolomics spectra

## Evidence
