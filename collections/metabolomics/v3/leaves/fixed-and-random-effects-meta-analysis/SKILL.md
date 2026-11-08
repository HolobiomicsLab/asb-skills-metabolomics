---
name: fixed-and-random-effects-meta-analysis
description: Use when the input is a per-study table of effect sizes with standard
  errors (one row per study per compound) and the goal is a pooled combined P-value
  per compound, choosing fixed vs random effects by I2 heterogeneity.
when_to_use_negative:
- Input studies report only P-values and fold-changes without standard deviations/errors
  — metabolomics studies rarely disclose this, so inverse-variance pooling is impossible;
  use amanida's weighted Fisher / weighted fold-change averaging instead
- Only qualitative trend labels (up/down-regulation) are available per compound —
  use vote-counting (amanida_vote) rather than effect-size pooling
- Data are individual-participant raw data needing a single unified model rather than
  aggregation of published per-study summary effects
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: meta
  role: R package providing metagen/metabin for fixed-effects inverse-variance meta-analysis,
    Cochran's Q, I2, and DerSimonian-Laird random-effects pooling
- name: amanida
  role: Alternative weighted meta-analysis tool (Fisher's method, fold-change averaging,
    vote-counting) used when the per-study standard errors required by 'meta' are
    not disclosed
  repo: github.com/mariallr/amanida
- name: R
  role: Execution environment in which the meta-analysis workflow (metagen/metabin
    calls, Q/I2 computation, results export) is run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:2c0c6bbbd49ad2c4f3acc0976308078730752fae36353123d5532e194bcba872
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
  - meta
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# fixed-and-random-effects-meta-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classical inverse-variance meta-analysis of per-study effect sizes using the 'meta' R package: fit a fixed-effects model, compute Cochran's Q and I2 heterogeneity, and switch to a DerSimonian-Laird random-effects model when I2 > 40%. Applied per compound (e.g. amino acids in lung-cancer data) to obtain pooled P-values.

## When to use

Use when the input is a per-study table of effect sizes with standard errors (one row per study per compound) and the goal is a pooled combined P-value per compound, choosing fixed vs random effects by I2 heterogeneity.

## When NOT to use

- Input studies report only P-values and fold-changes without standard deviations/errors — metabolomics studies rarely disclose this, so inverse-variance pooling is impossible; use amanida's weighted Fisher / weighted fold-change averaging instead
- Only qualitative trend labels (up/down-regulation) are available per compound — use vote-counting (amanida_vote) rather than effect-size pooling
- Data are individual-participant raw data needing a single unified model rather than aggregation of published per-study summary effects

## Inputs

- R data frame of per-study effect sizes and standard errors for each compound (lung cancer amino acid meta-analysis dataset)
- Compound/study identifiers and per-study sample data suitable for metagen (effect, SE) or metabin (binary counts) input

## Outputs

- meta_results.csv with per-compound columns: meta-analysis model applied (fixed vs random), combined P-value, and I2
- Significance calls per compound (e.g. Proline significant under the random-effect model)

## How to apply

Load the dataset of per-study effect sizes and standard errors (e.g. lung cancer amino acid data, one row per study per amino acid compound) into R and install the 'meta' package. For each compound, fit a fixed-effects inverse-variance model using metagen (continuous/log-ratio effects) or metabin (binary/outcome-count data) from the metagen/metabin function family. Compute Cochran's Q and the I2 heterogeneity statistic for each compound, and switch to a DerSimonian-Laird random-effects model whenever I2 > 40% (high heterogeneity invalidates the fixed-effect pooling assumption). Extract, per compound, the model applied, the combined P-value, and I2, and write them to meta_results.csv. Judge correctness by checking that the reported compound reproduces: Proline must be significant under the random-effects model with I2 = 87.1%.

## Related tools

- **meta** (R package providing metagen/metabin for fixed-effects inverse-variance meta-analysis, Cochran's Q, I2, and DerSimonian-Laird random-effects pooling)
- **amanida** (Alternative weighted meta-analysis tool (Fisher's method, fold-change averaging, vote-counting) used when the per-study standard errors required by 'meta' are not disclosed) — github.com/mariallr/amanida
- **R** (Execution environment in which the meta-analysis workflow (metagen/metabin calls, Q/I2 computation, results export) is run)

## Examples

```
library(meta); res <- metagen(TE = effect, seTE = se, studlab = study, data = aa_data); res$I2 > 40  # switch to comb.random = TRUE when I2 exceeds 40%
```

## Evaluation signals

- meta_results.csv contains, for every compound, the model type applied, the combined P-value, and the I2 value
- Every compound with I2 > 40% is analyzed under the DerSimonian-Laird random-effects model, and compounds with I2 <= 40% under fixed effects
- I2 values are consistent with the Cochran's Q statistic and degrees of freedom (number of studies - 1) per compound
- Reproduced check point: Proline is significant under the random-effect model with heterogeneity I2 = 87.1%
- Fixed-effects inverse-variance pooling was applied before any heterogeneity-based model switch (no unconditional random-effects default)

## Limitations

- Requires per-study effect sizes AND standard errors; metabolomics studies rarely disclose the statistical information needed, so many compound tables cannot support this classical approach
- Model choice depends on the I2 > 40% threshold — heterogeneity near the cutoff makes the fixed/random decision unstable
- Random-effects results (e.g. Proline at I2 = 87.1%) reflect wide between-study heterogeneity and rest on DerSimonian-Laird variance estimation, which can be unreliable with few studies

## Evidence

- [other] Compute Cochran's Q and the I2 heterogeneity statistic for each compound and switch to a DerSimonian-Laird random-effects model whenever I2 > 40%.: [evidence span withheld — anchor status: not_found]
- [results] Repeating the classical meta-analysis with the 'Meta' R package yielded significant results for Proline under the random-effect model, with heterogeneity I2 = 87.1%.: [evidence span withheld — anchor status: not_found]
- [other] fit a fixed-effects inverse-variance meta-analysis model using the 'meta' R package (metagen/metabin family of functions): [evidence span withheld — anchor status: not_found]
- [readme] It covers the lack of data provided on metabolomic studies, where is rare to have error or variance disclosed.: [evidence span withheld — anchor status: external_doc]
