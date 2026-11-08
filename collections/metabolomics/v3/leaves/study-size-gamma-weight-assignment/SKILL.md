---
name: study-size-gamma-weight-assignment
description: Use when combining per-study overall P-values from metabolomics studies
  where only P-value and fold-change (not variance or raw data) are reported, and
  each study's participant count (n) is available as the sole basis for weighting.
when_to_use_negative:
- Raw data, standard deviations, or variances are disclosed — conventional meta-analysis
  (e.g. inverse-variance weighting) is then applicable instead
- The input is trend labels only (up-/down-regulated) with no P-values — use amanida's
  qualitative vote-counting (amanida_vote) rather than weighted P-value combination
- Fold-change/effect-size combination is the goal alone — that uses a separate log-transformed
  weighted average in the same package, not the gamma-weighted Fisher step
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: amanida
  role: R package implementing the gamma-distributed non-integral weighting and the
    weighted Fisher's-method P-value combination inside compute_amanida
  repo: github.com/mariallr/amanida
- name: R
  role: Runtime environment in which the amanida weighted meta-analysis functions
    are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:dc77fa91f95d554129b6379aa210a26eb9f2526ebee804af832b19f1cee754bc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/amanida
  tools:
  - R
  - amanida
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# study-size-gamma-weight-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns non-integral per-study weights to metabolomics P-values via a gamma distribution proportional to study size (n), enabling a weighted Fisher's-method combination of overall results when raw data or standard deviations are not disclosed.

## When to use

Use when combining per-study overall P-values from metabolomics studies where only P-value and fold-change (not variance or raw data) are reported, and each study's participant count (n) is available as the sole basis for weighting.

## When NOT to use

- Raw data, standard deviations, or variances are disclosed — conventional meta-analysis (e.g. inverse-variance weighting) is then applicable instead
- The input is trend labels only (up-/down-regulated) with no P-values — use amanida's qualitative vote-counting (amanida_vote) rather than weighted P-value combination
- Fold-change/effect-size combination is the goal alone — that uses a separate log-transformed weighted average in the same package, not the gamma-weighted Fisher step

## Inputs

- Per-study overall P-values for each compound
- Per-study study size (N total, number of participants) per compound
- Dataset in csv/xls/xlsx/txt format readable by amanida_read with mode = "quan" and coln = c("Compound Name", "P-value", "Fold-change", "N total", "References")

## Outputs

- A single combined (weighted Fisher) P-value per compound
- A table of per-compound combined P-values, e.g. exported as sample_combined_pvalues.csv or accessible via amanida_result@stat

## How to apply

Load the compounds' per-study P-values together with each study's sample size (n), typically from a dataset read via amanida_read in mode = "quan" with columns ordered as Id, P-value, Fold-change, N, References. For each compound, compute a non-integral weight proportional to the study size by evaluating the corresponding gamma distribution (Section 2 of the paper), so larger cohorts contribute proportionally more. Apply the weighted Fisher's method — combining the statistic across studies with these gamma-assigned weights — to yield one combined P-value per compound, as implemented internally by compute_amanida. The rationale is that metabolomics studies rarely disclose the standard deviations or raw data required for conventional inverse-variance weighting, so study size becomes the only defensible weight.

## Related tools

- **amanida** (R package implementing the gamma-distributed non-integral weighting and the weighted Fisher's-method P-value combination inside compute_amanida) — github.com/mariallr/amanida
- **R** (Runtime environment in which the amanida weighted meta-analysis functions are executed)

## Examples

```
library(amanida); datafile <- amanida_read(input_file, mode="quan", coln=c("Compound Name","P-value","Fold-change","N total","References"), separator=";"); amanida_result <- compute_amanida(datafile, comp.inf = F)
```

## Evaluation signals

- Each compound in the output carries exactly one combined P-value derived from all of its contributing studies' reported P-values
- Weights are non-integral (not simple study counts) and scale monotonically with study size n, consistent with gamma-distribution assignment
- Running compute_amanida on the bundled sample_data reproduces the per-compound combined statistics obtainable from the Colorectal Cancer urine volatilome/metabolome review dataset
- Combined P-values should be usable downstream with a significance cutoff (e.g. pvalue_cutoff = 0.05) in amanida_report / volcano_plot without errors
- Cross-check that no compound required standard deviations — any case needing variance input signals misuse of this skill

## Limitations

- Requires every contributing study to report an overall P-value and a participant count n; metabolomics studies rarely disclose all statistical information needed, so compounds lacking either cannot be weighted this way
- Study size is used as the weighting basis because variance/SD is unavailable — this discards any within-study precision information
- Identifiers must be curated/harmonized beforehand (e.g. via check_names to PubChem IDs under the Metabolites Merging Strategy) so per-compound P-value groups are correctly assembled

## Evidence

- [other] Gamma-assigned non-integral weights proportional to study size: [evidence span withheld — anchor status: not_found]
- [readme] README: Fisher weighting by participants: [evidence span withheld — anchor status: external_doc]
