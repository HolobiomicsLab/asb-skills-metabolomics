---
name: clinical-metabolomics-dataset-curation
description: Use when a task requires combining aggregate statistics (p-value, fold-change)
  from multiple clinical metabolomics studies where compound names differ across sources
  and per-study raw data are unavailable.
when_to_use_negative:
- Input data are raw metabolomics spectra or peak tables rather than published aggregate
  statistics (p-value, fold-change) per compound.
- The analysis goal is single-study differential analysis rather than combining results
  across multiple studies.
- Compound identity must be matched strictly by database IDs and the 'Include compounds
  IDs' option is required but ID mapping is unavailable.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Easy-Amanida
  role: Shiny web app providing the workflow to upload aggregate-statistics tables,
    optimize the naming combination, and run the meta-analysis
  repo: github.com/mariallr/easy-amanida
- name: amanida
  role: R package that computes the weighted combination of p-values and log2 fold-changes
    per compound
  repo: github.com/mariallr/easy-amanida
- name: webchem
  role: R package backing the compound naming harmonization that unifies synonyms
    across studies
- name: Shiny
  role: R add-on used to implement the Easy-Amanida interactive web app
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:eb9429b939c71bc30bfd5e76c0d8b479431baf4d57510e1b204b6f2d45306aa1
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# clinical-metabolomics-dataset-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curate aggregate metabolomics statistics (p-values and fold-changes) from multiple clinical studies and harmonize compound names so they can be combined in an Amanida-based meta-analysis, as implemented in the Easy-Amanida Shiny app.

## When to use

Use when a task requires combining aggregate statistics (p-value, fold-change) from multiple clinical metabolomics studies where compound names differ across sources and per-study raw data are unavailable.

## When NOT to use

- Input data are raw metabolomics spectra or peak tables rather than published aggregate statistics (p-value, fold-change) per compound.
- The analysis goal is single-study differential analysis rather than combining results across multiple studies.
- Compound identity must be matched strictly by database IDs and the 'Include compounds IDs' option is required but ID mapping is unavailable.

## Inputs

- CSV/Excel table of compounds with per-study aggregate statistics (p-value and fold-change), e.g. PDAC Supplementary Table 1 from 24 clinical studies
- Optionally a compounds ID list (unchecked by default in Easy-Amanida)

## Outputs

- Per-compound combined meta-analysis results table (e.g. amanida_metaanalysis_results.csv) with combined p-values and log2 fold-changes
- Counts of combined metabolites, multi-study compounds, and significant hits (p<0.05, FC>2)

## How to apply

Load a table of aggregate statistics (p-value and fold-change) per compound per study, e.g. PDAC Supplementary Table 1 spanning 24 clinical studies, into the Easy-Amanida Shiny app or the underlying amanida R package. Leave the 'Include compounds IDs' option unchecked so compounds are matched by name, and rely on the webchem-backed naming harmonization to unify compound synonyms before combining. Run the amanida meta-analysis to compute the weighted combination of p-values and log2 fold-changes per compound, producing a per-compound combined result table that can be exported to amanida_metaanalysis_results.csv. Judge correctness by reproducing reported anchors: 311 metabolites combined, 38 compounds appearing in more than one study, 35 significant metabolites (p<0.05 and FC>2), and Table 2 entries such as Histidine combined p-value 0.023 (N 9500), Valine FC 160 up-regulated, and Tryptophan FC 150 up-regulated.

## Related tools

- **Easy-Amanida** (Shiny web app providing the workflow to upload aggregate-statistics tables, optimize the naming combination, and run the meta-analysis) — github.com/mariallr/easy-amanida
- **amanida** (R package that computes the weighted combination of p-values and log2 fold-changes per compound) — github.com/mariallr/easy-amanida
- **webchem** (R package backing the compound naming harmonization that unifies synonyms across studies)
- **Shiny** (R add-on used to implement the Easy-Amanida interactive web app)

## Examples

```
Upload PDAC Supplementary Table 1 (p-value, fold-change per study) at http://brui.shinyapps.io/easy-amanida, leave 'Include compounds IDs' unchecked, run amanida meta-analysis, export amanida_metaanalysis_results.csv
```

## Evaluation signals

- Output combines 311 metabolites for the PDAC dataset of 24 clinical studies
- 38 compounds appear in more than one study after name harmonization
- 35 metabolites are flagged significant with combined p<0.05 and fold-change >2
- Table 2 anchors reproduce: Histidine combined p-value 0.023 with N 9500; Valine FC 160 up-regulated; Tryptophan FC 150 up-regulated
- Exported amanida_metaanalysis_results.csv contains one combined result row per harmonized compound

## Limitations

- Works on aggregate statistics (p-value, fold-change), so it cannot reanalyze per-sample raw data.
- Compound matching depends on naming harmonization via webchem; unresolved synonyms may split or merge compounds incorrectly.
- Metabolomics meta-analysis is hindered by wide compound variety, differing experimental methods, and lack of consensus on statistical reporting across studies.

## Evidence
