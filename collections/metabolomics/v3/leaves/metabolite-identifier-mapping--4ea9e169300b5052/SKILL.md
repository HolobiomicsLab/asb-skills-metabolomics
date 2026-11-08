---
name: metabolite-identifier-mapping
description: Use when combining metabolomics studies that report the same metabolite
  under different names/synonyms, and compound-level aggregation (e.g. vote-counting
  in amanida) would otherwise split or mismatch hits.
when_to_use_negative:
- Input is raw spectral data (e.g. mzML/NetCDF peaks) rather than a list of compounds
  with aggregate statistics
- Compound names are already reported under a single consistent nomenclature across
  all studies, so harmonization adds no value
- The meta-analysis operates on non-metabolite features (e.g. genes or proteins) outside
  webchem's compound-mapping scope
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: webchem
  role: Provides compound naming harmonization, mapping study-specific metabolite
    synonyms to standard identifiers before meta-analysis
- name: Easy-Amanida
  role: Shiny web app that embeds the harmonization workflow to optimize naming combination
    prior to amanida meta-analysis
  repo: https://github.com/mariallr/easy-amanida
- name: amanida
  role: Consumes the harmonized compound table to perform vote-counting and p-value/fold-change
    meta-analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:fe8559477cfeede64be811b5f833bd39fc453d39d9c19294abcb00a4cfa38d30
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
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# metabolite-identifier-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes heterogeneous metabolite compound names across studies before meta-analysis of aggregate statistics (p-values, fold-changes), using the webchem package inside the Easy-Amanida Shiny app so that votes and pooled statistics are aggregated per unique compound rather than per name variant.

## When to use

Use when combining metabolomics studies that report the same metabolite under different names/synonyms, and compound-level aggregation (e.g. vote-counting in amanida) would otherwise split or mismatch hits.

## When NOT to use

- Input is raw spectral data (e.g. mzML/NetCDF peaks) rather than a list of compounds with aggregate statistics
- Compound names are already reported under a single consistent nomenclature across all studies, so harmonization adds no value
- The meta-analysis operates on non-metabolite features (e.g. genes or proteins) outside webchem's compound-mapping scope

## Inputs

- Table of compound names with per-study aggregate statistics (p-value, fold-change), e.g. CSV from the easy-amanida GitHub repository
- List of compound name synonyms/variants across studies

## Outputs

- Harmonized compound table with standardized identifiers mapped via webchem
- Combined meta-analysis input suitable for amanida vote-counting (e.g. vote_counting_results.csv)

## How to apply

Load the input table of compounds with aggregate statistics (p-value and fold-change per study) into the Easy-Amanida Shiny app (or call amanida directly in R). Run the compound naming harmonization step, which uses webchem to map synonym/variant compound names onto standardized identifiers, providing a workflow to optimize the naming combination so the same metabolite from different studies is collapsed to one entry. Only after harmonization should the amanida meta-analysis (e.g. qualitative vote-counting assigning +1 for up-regulated and -1 for down-regulated per study) be applied, since unharmonized names would fragment recurrence counts such as glutamine appearing in 7 studies. Inspect the harmonization output to resolve ambiguous or unmatched names before aggregating, and write the resulting combined table (e.g. vote_counting_results.csv).

## Related tools

- **webchem** (Provides compound naming harmonization, mapping study-specific metabolite synonyms to standard identifiers before meta-analysis)
- **Easy-Amanida** (Shiny web app that embeds the harmonization workflow to optimize naming combination prior to amanida meta-analysis) — https://github.com/mariallr/easy-amanida
- **amanida** (Consumes the harmonized compound table to perform vote-counting and p-value/fold-change meta-analysis)

## Examples

```
shiny::runGitHub("mariallr/easy-amanida")  # launch app, upload compounds CSV with p-values and fold-changes, run naming harmonization then amanida vote-counting
```

## Evaluation signals

- No duplicate or split entries remain: each metabolite reported under multiple synonyms is collapsed to a single harmonized identifier
- Downstream vote counts match expectations after harmonization, e.g. glutamine recurrent in 7 studies, histidine in 5, tauroursodeoxycholic/taurocholic/glycocholic acid in 3 each
- The 15 metabolites with inconsistent regulation trends are flagged and excluded, leaving 8 up- and 12 down-regulated significant compounds
- Ambiguous or unmatched compound mappings are surfaced for manual review rather than silently dropped
- Output table (e.g. vote_counting_results.csv) carries both the harmonized identifier and the original study-level statistics

## Limitations

- Harmonization depends on webchem identifier lookups; compounds absent from reference databases or reported only as abbreviations may fail to map and require manual curation
- The wide variety of compounds and differing experimental methods in metabolomics, plus lack of consensus on statistical reporting, make harmonization challenging and error-prone
- Easy-Amanida targets aggregate statistical data (p-value, fold-change); it cannot harmonize or meta-analyze raw spectra or individual-level data

## Evidence
