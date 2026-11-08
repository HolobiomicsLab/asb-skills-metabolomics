---
name: log-fold-change-pooling
description: Use when contributing studies report only aggregate statistics (fold-change
  plus sample size, no raw data) and a per-compound combined effect direction and
  magnitude must be derived.
when_to_use_negative:
- Studies provide raw individual-level data, where direct meta-analysis of raw measurements
  is preferable to pooling aggregate fold-changes
- Fold-change direction or sample size is missing for a study, since weights are proportional
  to sample size and sign determines regulation direction
- The goal is compound naming harmonization or identifier mapping rather than combining
  effect sizes (use the naming-combination workflow instead)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: amanida
  role: R package implementing the weighted mean of log2-transformed fold-changes
    with weights proportional to study sample size
  repo: https://github.com/mariallr/easy-amanida
- name: Easy-Amanida
  role: Shiny web app exposing the Amanida fold-change pooling workflow over aggregate
    statistics tables
  repo: https://github.com/mariallr/easy-amanida
- name: R
  role: Runtime environment used to load the aggregate-statistics demo dataset and
    run the Amanida combination
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:3fedc6118375d022877e26d436cfc727aae636516b175aafae5be5bdf87bf7ab
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# log-fold-change-pooling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pools per-study fold-changes into a single combined fold-change by first log2-transforming each study's fold-change and then computing a weighted mean with weights proportional to each study's sample size, as implemented in the Amanida R package used by Easy-Amanida.

## When to use

Use when contributing studies report only aggregate statistics (fold-change plus sample size, no raw data) and a per-compound combined effect direction and magnitude must be derived.

## When NOT to use

- Studies provide raw individual-level data, where direct meta-analysis of raw measurements is preferable to pooling aggregate fold-changes
- Fold-change direction or sample size is missing for a study, since weights are proportional to sample size and sign determines regulation direction
- The goal is compound naming harmonization or identifier mapping rather than combining effect sizes (use the naming-combination workflow instead)

## Inputs

- Per-study aggregate statistics table (compound identifier, p-value, fold-change, sample size per study) from the demo dataset in the easy-amanida GitHub repository
- Harmonized compound groupings so rows referring to the same compound are grouped

## Outputs

- Combined (weighted mean) log2 fold-change per compound
- Regulation direction (up-regulated / down-regulated) per compound derived from the sign of the combined log2 fold-change
- amanida_meta_analysis_results.csv with combined p-value, combined fold-change, and regulation direction per compound

## How to apply

Collect per-study aggregate statistics (compound identifier, fold-change, sample size) into one table. Logarithmically transform each study's fold-change to base 2 so effects become symmetric around 0 and additive on the log scale. For each harmonized compound, compute the weighted mean of the log2 fold-changes, with weights proportional to each study's sample size, so larger studies contribute more. Classify the compound as up-regulated or down-regulated from the sign of the combined log2 fold-change. Write the combined fold-change and regulation direction per compound to amanida_meta_analysis_results.csv alongside the weighted-Fisher combined p-value.

## Related tools

- **amanida** (R package implementing the weighted mean of log2-transformed fold-changes with weights proportional to study sample size) — https://github.com/mariallr/easy-amanida
- **Easy-Amanida** (Shiny web app exposing the Amanida fold-change pooling workflow over aggregate statistics tables) — https://github.com/mariallr/easy-amanida
- **R** (Runtime environment used to load the aggregate-statistics demo dataset and run the Amanida combination)

## Examples

```
library(amanida); results <- amanida(data = aggregate_stats, typein = "raw", statistiq = TRUE); write.csv(results, "amanida_meta_analysis_results.csv")
```

## Evaluation signals

- Combined log2 fold-change equals the sample-size-weighted mean of the per-study log2 fold-changes for each compound group
- Sign of the combined log2 fold-change matches the assigned up-regulated / down-regulated direction
- Combined fold-change is reported on the log2 scale (transformed from linear fold-changes with base 2)
- amanida_meta_analysis_results.csv contains one row per compound with combined p-value, combined fold-change, and regulation direction
- Compounds with larger contributing studies show combined fold-changes pulled toward those studies' log2 fold-changes

## Limitations

- Method requires each contributing study to disclose aggregate statistics; studies reporting only p-values without fold-change cannot contribute to the fold-change pooling
- Compound naming synonyms must be harmonized first, or rows for the same compound will not be grouped and pooled correctly
- Fold-changes are logarithmically transformed with base 2; fold-changes given on a different scale or as non-symmetric ratios must be converted before pooling

## Evidence

- [other] Per-study p-values are combined using a weighted Fisher's method, with a gamma distribution assigning non-integral weights to each p-value proportional to study size; fold-changes are logarithmically transformed (base 2).: [evidence span withheld — anchor status: not_found]
