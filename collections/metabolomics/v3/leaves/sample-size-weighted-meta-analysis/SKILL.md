---
name: sample-size-weighted-meta-analysis
description: Use when contributing studies report only aggregate statistics (p-value,
  fold-change, sample size) per compound and you need to combine them into a single
  combined p-value, fold-change, and regulation direction per compound.
when_to_use_negative:
- Input studies provide individual-level (raw) data rather than aggregate statistics
  such as p-value and fold-change
- Studies report effect sizes other than p-value and fold-change (e.g. only qualitative
  direction calls)
- Compound names cannot be harmonized because no identifier mapping or naming-combination
  step can resolve synonyms across studies
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: amanida
  role: R package implementing the weighted Fisher's method for combining p-values
    and sample-size-weighted fold-change combination
  repo: github.com/mariallr/easy-amanida
- name: Easy-Amanida
  role: Shiny web app that wraps amanida and provides a workflow to optimize naming
    combination before meta-analysis
  repo: github.com/mariallr/easy-amanida
- name: webchem
  role: R package combined with amanida in Easy-Amanida to enable compounds naming
    harmonization
  repo: github.com/mariallr/easy-amanida
- name: R
  role: Language in which the amanida package and Easy-Amanida app are implemented
    and run
- name: Shiny
  role: R package add-on for interactive web apps in which Easy-Amanida is implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:a4770a16d9fb823516b4ae1dd75c9c1cca7528579306da732545351cbfa74fd9
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# sample-size-weighted-meta-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines per-study aggregate statistics (p-values and log2 fold-changes) across studies of the same compound using Amanida's weighted Fisher's method with non-integral gamma-distributed weights and sample-size-weighted averaging of fold-changes, as implemented in the amanida R package and Easy-Amanida Shiny app.

## When to use

Use when contributing studies report only aggregate statistics (p-value, fold-change, sample size) per compound and you need to combine them into a single combined p-value, fold-change, and regulation direction per compound.

## When NOT to use

- Input studies provide individual-level (raw) data rather than aggregate statistics such as p-value and fold-change
- Studies report effect sizes other than p-value and fold-change (e.g. only qualitative direction calls)
- Compound names cannot be harmonized because no identifier mapping or naming-combination step can resolve synonyms across studies

## Inputs

- Per-study aggregate statistics table with columns: compound identifier, p-value, fold-change, sample size per study (e.g. demo dataset from the easy-amanida GitHub repository)

## Outputs

- amanida_meta_analysis_results.csv containing combined p-value, combined fold-change (log2), and regulation direction (up-/down-regulated) per compound

## How to apply

Load the per-study aggregate statistics table (compound identifier, p-value, fold-change, sample size per study), e.g. from the demo dataset in the easy-amanida GitHub repository, using R. First harmonize compound identifiers across studies so rows referring to the same compound are grouped (Amanida naming-combination workflow, which can be optimized in Easy-Amanida). For each compound, combine per-study p-values with Amanida's weighted Fisher's method, in which a gamma distribution assigns non-integral weights to each p-value proportional to study size. For each compound, compute the weighted mean of log2-transformed (base 2) fold-changes, with weights proportional to study sample size. Classify each compound as up- or down-regulated from the sign of the combined log2 fold-change, and write combined p-value, combined fold-change, and regulation direction per compound to amanida_meta_analysis_results.csv.

## Related tools

- **amanida** (R package implementing the weighted Fisher's method for combining p-values and sample-size-weighted fold-change combination) — github.com/mariallr/easy-amanida
- **Easy-Amanida** (Shiny web app that wraps amanida and provides a workflow to optimize naming combination before meta-analysis) — github.com/mariallr/easy-amanida
- **webchem** (R package combined with amanida in Easy-Amanida to enable compounds naming harmonization) — github.com/mariallr/easy-amanida
- **R** (Language in which the amanida package and Easy-Amanida app are implemented and run)
- **Shiny** (R package add-on for interactive web apps in which Easy-Amanida is implemented)

## Examples

```
Rscript -e 'amanida::tote(aggregate_stats, type="meta", pval="pvalue", fold="fold-change")'
```

## Evaluation signals

- Combined p-values per compound match Amanida's weighted Fisher's method with gamma-distributed non-integral weights proportional to study size
- Combined fold-changes are reported on log2 scale and equal the sample-size-weighted mean of log2-transformed per-study fold-changes
- Rows referring to the same compound are grouped under harmonized identifiers (no duplicate compound entries under different synonyms)
- Regulation direction per compound matches the sign of the combined log2 fold-change
- Output file amanida_meta_analysis_results.csv contains combined p-value, combined fold-change, and direction for every compound in the input table

## Limitations

- Meta-analyses in metabolomics are hindered by the wide variety of compounds, different experimental methods, lack of consensus on statistical reporting, and compound naming synonyms
- Method applies to aggregate statistics only; individual-level data require a different approach
- Naming harmonization is error-prone because of compound naming synonyms, hence the need for a workflow to optimize naming combination

## Evidence

- [full_text] weighted Fisher's method with gamma weights: [evidence span withheld — anchor status: not_found]
