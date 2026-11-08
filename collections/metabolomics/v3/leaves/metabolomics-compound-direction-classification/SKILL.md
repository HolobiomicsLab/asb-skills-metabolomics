---
name: metabolomics-compound-direction-classification
description: Use when contributing studies report only aggregate statistics (p-value,
  fold-change, sample size) per compound and you need a per-compound regulation direction
  label from the meta-analyzed log2 fold-change.
when_to_use_negative:
- Input is raw or individual-level metabolomics intensity data rather than reported
  aggregate statistics (p-value, fold-change) — Easy-Amanida/Amanida is designed for
  aggregate results
- Studies report p-values but no fold-changes, so a regulation direction cannot be
  inferred from the sign of a combined log2 fold-change
- Compound identifiers have not been harmonized across studies (synonyms unresolved),
  so rows for the same compound would not be grouped before combination
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: amanida
  role: R package implementing the weighted Fisher's method for combining p-values
    and the log2 fold-change weighted averaging from which the regulation direction
    is derived
  repo: github.com/mariallr/easy-amanida
- name: Easy-Amanida
  role: Shiny web app wrapping amanida and webchem to run the aggregate-statistics
    meta-analysis and naming harmonization workflow end-to-end
  repo: github.com/mariallr/easy-amanida
- name: webchem
  role: Supports compound naming harmonization so that rows for the same compound
    are grouped before direction classification
  repo: github.com/mariallr/easy-amanida
- name: R
  role: Language in which the amanida combination and classification are executed
    (or via the Shiny app)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:87126e15643bdcfe45c2229bb32a09f5289fa7ddcc842131f5a2b67fb0b7d71b
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
  - amanida
  - webchem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolomics-compound-direction-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies each compound in an Amanida/Easy-Amanida aggregate-statistics meta-analysis as up- or down-regulated based on the sign of its combined log2-transformed fold-change, after p-values are combined by weighted Fisher's method and fold-changes by sample-size-weighted averaging.

## When to use

Use when contributing studies report only aggregate statistics (p-value, fold-change, sample size) per compound and you need a per-compound regulation direction label from the meta-analyzed log2 fold-change.

## When NOT to use

- Input is raw or individual-level metabolomics intensity data rather than reported aggregate statistics (p-value, fold-change) — Easy-Amanida/Amanida is designed for aggregate results
- Studies report p-values but no fold-changes, so a regulation direction cannot be inferred from the sign of a combined log2 fold-change
- Compound identifiers have not been harmonized across studies (synonyms unresolved), so rows for the same compound would not be grouped before combination

## Inputs

- Per-study aggregate statistics table: compound identifier, p-value, fold-change, sample size per study (demo dataset from mariallr/easy-amanida)
- Harmonized compound identifier mapping across studies (amanida/webchem naming combination)

## Outputs

- amanida_meta_analysis_results.csv with combined p-value, combined fold-change, and up-/down-regulation direction per compound

## How to apply

Load the per-study aggregate statistics table (compound identifier, p-value, fold-change, sample size) from the demo dataset in the mariallr/easy-amanida repository and harmonize compound identifiers across studies so rows for the same compound are grouped (the amanida naming-combination workflow, with webchem-backed harmonization). For each compound, combine p-values with Amanida's weighted Fisher's method, where non-integral weights are drawn from a gamma distribution and are proportional to each study's sample size. Compute the weighted mean of log2-transformed (base 2) fold-changes, again with weights proportional to study sample size. Assign the regulation direction from the sign of the combined log2 fold-change: positive sign → up-regulated, negative sign → down-regulated. Write the combined p-value, combined fold-change, and direction per compound to amanida_meta_analysis_results.csv.

## Related tools

- **amanida** (R package implementing the weighted Fisher's method for combining p-values and the log2 fold-change weighted averaging from which the regulation direction is derived) — github.com/mariallr/easy-amanida
- **Easy-Amanida** (Shiny web app wrapping amanida and webchem to run the aggregate-statistics meta-analysis and naming harmonization workflow end-to-end) — github.com/mariallr/easy-amanida
- **webchem** (Supports compound naming harmonization so that rows for the same compound are grouped before direction classification) — github.com/mariallr/easy-amanida
- **R** (Language in which the amanida combination and classification are executed (or via the Shiny app))

## Evaluation signals

- Each compound's regulation label matches the sign of its combined log2 fold-change (positive → up-regulated, negative → down-regulated)
- Combined fold-change equals the weighted mean of base-2 log-transformed per-study fold-changes with weights proportional to study sample size
- Combined p-value falls in (0, 1] and reproduces Amanida's gamma-weighted Fisher combination for the same per-study p-values
- Output file amanida_meta_analysis_results.csv contains, per compound identifier, the combined p-value, combined fold-change, and direction
- Compounds referenced under synonyms in different studies are grouped into a single row after the naming-combination workflow

## Limitations

- Metabolomics meta-analysis is hindered by wide compound variety, differing experimental methods, lack of consensus on statistical reporting, and compound naming synonyms — mis-grouped identifiers corrupt the per-compound direction
- Applicable only when studies disclose aggregate statistics (p-value and fold-change); direction cannot be classified for compounds lacking a fold-change
- Discussion-section content of the source article was unavailable, so additional caveats (e.g. changelog notes) could not be extracted

## Evidence
