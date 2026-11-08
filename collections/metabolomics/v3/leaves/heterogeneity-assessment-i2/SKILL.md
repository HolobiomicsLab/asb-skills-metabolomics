---
name: heterogeneity-assessment-i2
description: Use when reproducing or conducting a classical meta-analysis on per-study
  effect sizes and standard errors, and you must decide per compound between fixed-effect
  and random-effects (DerSimonian-Laird) models based on an I² heterogeneity threshold.
when_to_use_negative:
- Raw data or effect sizes with standard deviations/variances are unavailable — amanida's
  weighted Fisher/vote-counting approach is designed for non-integral P-value + fold-change
  data, not inverse-variance models
- Only qualitative trend labels are reported per compound — vote-counting, not Q/I²-based
  model selection, is appropriate
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3570
tools:
- name: meta
  role: R package (metagen/metabin functions) for fitting fixed-effects inverse-variance
    models and computing Cochran's Q and I² to select fixed vs. random-effects models
- name: R
  role: Computing environment for the meta-analysis workflow and writing meta_results.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:c39313a64b532f14341942bb26a8b126e2175d993a88b3bd769ddcb512315230
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
  - meta
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3570
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# heterogeneity-assessment-i2

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess between-study heterogeneity in classical inverse-variance meta-analysis by computing Cochran's Q and I² per compound in the 'meta' R package, and switch to a DerSimonian-Laird random-effects model whenever I² > 40%. Used to reproduce metabolomics meta-analysis results such as Proline significance (I² = 87.1%).

## When to use

Use when reproducing or conducting a classical meta-analysis on per-study effect sizes and standard errors, and you must decide per compound between fixed-effect and random-effects (DerSimonian-Laird) models based on an I² heterogeneity threshold.

## When NOT to use

- Raw data or effect sizes with standard deviations/variances are unavailable — amanida's weighted Fisher/vote-counting approach is designed for non-integral P-value + fold-change data, not inverse-variance models
- Only qualitative trend labels are reported per compound — vote-counting, not Q/I²-based model selection, is appropriate

## Inputs

- Per-study meta-analysis dataset of effect sizes and standard errors per compound (e.g. lung cancer amino acid data), loaded into R
- Compound identifiers grouping the per-study estimates

## Outputs

- meta_results.csv listing per-compound model applied (fixed vs. DerSimonian-Laird random-effects), combined P-value, and I²
- Cochran's Q statistic and I² heterogeneity estimates per compound

## How to apply

Load the dataset of per-study effect sizes and standard errors for each amino acid compound into R. For each compound, fit a fixed-effects inverse-variance meta-analysis model using the 'meta' package (metagen/metabin family of functions), then compute Cochran's Q and the I² heterogeneity statistic. Apply the decision rule: switch to a DerSimonian-Laird random-effects model whenever I² > 40%. Extract the model applied, the combined P-value, and I² for each compound into meta_results.csv, and verify that replicated findings match reported statistics (e.g. Proline significant under the random-effects model with I² = 87.1%).

## Related tools

- **meta** (R package (metagen/metabin functions) for fitting fixed-effects inverse-variance models and computing Cochran's Q and I² to select fixed vs. random-effects models)
- **R** (Computing environment for the meta-analysis workflow and writing meta_results.csv)

## Examples

```
library(meta); m <- metagen(TE, seTE, studlab, data = dat, sm = "SMD"); if (m$I2 > 40) m <- update(m, method.tau = "DL", random = TRUE); write.csv(data.frame(compound, m$model, m$pval.random, m$I2), "meta_results.csv")
```

## Evaluation signals

- Proline is significant under the random-effects model with reported heterogeneity I² = 87.1%
- Every compound with I² ≤ 40% is analyzed under the fixed-effects inverse-variance model; compounds with I² > 40% are switched to DerSimonian-Laird random-effects
- meta_results.csv contains, for every compound, the model applied, combined P-value, and I² with no missing entries
- Combined statistics reproduce the reported statistically significant result for Proline from the classical Lee et al. meta-analysis

## Limitations

- Heterogeneity is high for some metabolomics compounds (e.g. I² = 87.1% for Proline), making the random-effects model mandatory and widening confidence intervals
- Metabolomics studies rarely disclose all statistical information needed to perform a meta-analysis, limiting the applicability of classical effect-size/SE meta-analysis

## Evidence

- [other] Compute Cochran's Q and the I2 heterogeneity statistic for each compound and switch to a DerSimonian-Laird random-effects model whenever I2 > 40%.: [evidence span withheld — anchor status: not_found]
- [results] Repeating the classical meta-analysis with the 'Meta' R package yielded significant results for Proline under the random-effect model, with heterogeneity I2 = 87.1%.: [evidence span withheld — anchor status: not_found]
- [methods] For each compound, fit a fixed-effects inverse-variance meta-analysis model using the 'meta' R package (metagen/metabin family of functions).: [evidence span withheld — anchor status: not_found]
