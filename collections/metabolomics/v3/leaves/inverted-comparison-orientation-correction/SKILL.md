---
name: inverted-comparison-orientation-correction
description: Use when pooling fold-changes from multiple metabolomics studies where
  some reports give the inverted ratio (control/case instead of case/control), and
  a case-vs-control-consistent combined estimate per compound is required.
when_to_use_negative:
- When raw intensity data and standard deviations are available — use standard effect-size
  meta-analysis instead of amanida's fold-change-only weighted average
- When the input is trend/behaviour labels only (qualitative data) — use amanida's
  vote-counting, not fold-change orientation correction
- When identifiers have not been curated/harmonized — amanida requires identifier
  curation and PubChem ID harmonization (check_names) before meta-analysis
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: amanida
  role: R package whose quantitative meta-analysis combines log2-transformed fold-changes
    by study-size-weighted averaging; orientation correction is a preprocessing step
    feeding its compute_amanida input
  repo: https://github.com/mariallr/amanida
- name: R
  role: Execution environment for loading sample_data, applying -1/value reversal
    and log2 transformation, and running amanida functions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:e4d388c18772b45cdc83b93d624b77cc5bd78c9bbe0c2c6f59ac06713999aadc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/amanida
  repo_url: https://github.com/mariallr/amanida
  tools:
  - R
  - amanida
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# inverted-comparison-orientation-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Before combining fold-changes across metabolomics studies in amanida, detect studies that report the inverse comparison orientation (control/case) and reverse their fold-change values via -1/value, so that all per-compound values share the case-vs-control direction prior to log2 transformation and study-size-weighted averaging.

## When to use

Use when pooling fold-changes from multiple metabolomics studies where some reports give the inverted ratio (control/case instead of case/control), and a case-vs-control-consistent combined estimate per compound is required.

## When NOT to use

- When raw intensity data and standard deviations are available — use standard effect-size meta-analysis instead of amanida's fold-change-only weighted average
- When the input is trend/behaviour labels only (qualitative data) — use amanida's vote-counting, not fold-change orientation correction
- When identifiers have not been curated/harmonized — amanida requires identifier curation and PubChem ID harmonization (check_names) before meta-analysis

## Inputs

- Per-study fold-change table per compound (csv/xls/xlsx/txt read via amanida_read with mode = "quan"), with columns: Id, P-value, Fold-change, N (study size), Reference
- Flag or annotation identifying which fold-changes are control/case (inverted) comparisons

## Outputs

- Per-compound table of oriented, log2-transformed fold-changes combined by study-size-weighted averaging (e.g. combined_fold_changes.csv)
- amanida S4 result object (amanida_result@stat) containing combined meta-analysis effect sizes

## How to apply

Inspect each study's reported fold-change per compound and identify entries flagged as inverted comparisons (control/case orientation). For those entries apply -1/value, e.g. a fold-change of 0.5 reported as control/case becomes -2, restoring the case-vs-control direction; leave correctly oriented values unchanged. Leave missing fold-change entries as NA and skip them rather than imputing zero, since amanida ignores missing data. Then log2-transform the oriented fold-changes — the log transform reduces skewness and makes up- and down-regulation symmetric — and average per compound with weights proportional to study size (n), as in equation (2) of the amanida paper. Write the resulting per-compound combined fold-changes (e.g. to combined_fold_changes.csv) and check that inverted-but-reversed studies no longer pull the combined estimate in the wrong direction.

## Related tools

- **amanida** (R package whose quantitative meta-analysis combines log2-transformed fold-changes by study-size-weighted averaging; orientation correction is a preprocessing step feeding its compute_amanida input) — https://github.com/mariallr/amanida
- **R** (Execution environment for loading sample_data, applying -1/value reversal and log2 transformation, and running amanida functions)

## Examples

```
library(amanida); data("sample_data"); amanida_result <- compute_amanida(datafile, comp.inf = F)
```

## Evaluation signals

- Every entry annotated as control/case has been transformed by -1/value (e.g. 0.5 → -2), and no non-inverted entry was altered
- Missing fold-changes remain NA and are skipped, never replaced by 0, before averaging
- Reversing the reversal (re-applying -1/value) recovers the original reported values — an involutive sanity check
- Combined per-compound fold-changes in combined_fold_changes.csv match a manual recomputation of the n-weighted mean of log2 values for at least one compound
- Studies reporting an inverted comparison no longer flip the sign/direction of the combined case-vs-control estimate for their compounds

## Limitations

- Metabolomics studies rarely disclose all statistical information needed for meta-analysis, so fold-changes may be missing or inconsistently oriented across reports
- The method requires only p-value and fold-change and no standard deviation, so per-study variance cannot be accounted for in the combination
- Identifier harmonization (conversion to PubChem ID via check_names/webchem) must precede meta-analysis, otherwise duplicate compound entries are not detected

## Evidence

- [methods] Fold-changes are base-2 log-transformed to reduce skewness and averaged with weights proportional to study size; missing data are ignored, and negative fold-change values standing for inverse comparisons (control/case) are reversed via -1/value.: [evidence span withheld — anchor status: not_found]
- [readme] Fold-change combination: logarithmic transformation for average with weighting by number of participants.: [evidence span withheld — anchor status: external_doc]
