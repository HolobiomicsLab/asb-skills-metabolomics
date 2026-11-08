---
name: amino-acid-metabolomics-meta-analysis-comparison
description: Use when you have a table of per-study effect sizes and standard errors
  for multiple metabolite/amino acid compounds and need to pool them per compound,
  test heterogeneity (Cochran's Q, I2), and reproduce or cross-check a published significant
  compound-level result.
when_to_use_negative:
- 'Input lacks per-study variances/standard deviations — metabolomics studies rarely
  disclose them; use a non-integral weighted approach (amanida: P-value + fold-change)
  instead'
- Only qualitative trend labels (up/down-regulation) per study are available — vote-counting,
  not inverse-variance pooling, is appropriate
- A single study per compound — no pooling or heterogeneity estimate is possible
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: meta
  role: R package used to fit fixed-effects inverse-variance and DerSimonian-Laird
    random-effects models per compound (metagen/metabin functions) and to compute
    Cochran's Q and I2
- name: R
  role: Analysis environment in which the compound-wise meta-analysis loop, threshold
    switching, and meta_results.csv export are scripted
- name: amanida
  role: Alternative R package for the same metabolomics meta-analysis goal when only
    significance and fold-change (non-integral data) are reported
  repo: https://github.com/mariallr/amanida
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:95d63d19263c08488f33623cf47ed13637f4abb89d2d5fdf3ea5f4d2e54737cf
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
  - meta
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# amino-acid-metabolomics-meta-analysis-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce a classical inverse-variance meta-analysis over per-study effect sizes of amino acid/metabolite compounds in R with the 'meta' package, applying a fixed-effects model and switching to DerSimonian-Laird random effects when heterogeneity I2 exceeds 40%, so results (e.g. Proline significance) can be replicated and compared.

## When to use

Use when you have a table of per-study effect sizes and standard errors for multiple metabolite/amino acid compounds and need to pool them per compound, test heterogeneity (Cochran's Q, I2), and reproduce or cross-check a published significant compound-level result.

## When NOT to use

- Input lacks per-study variances/standard deviations — metabolomics studies rarely disclose them; use a non-integral weighted approach (amanida: P-value + fold-change) instead
- Only qualitative trend labels (up/down-regulation) per study are available — vote-counting, not inverse-variance pooling, is appropriate
- A single study per compound — no pooling or heterogeneity estimate is possible

## Inputs

- Per-compound dataset of per-study effect sizes and standard errors (e.g. csv/xlsx loaded into R) for each amino acid compound
- Optional: per-study sample sizes and trend labels for the non-integral (P-value/fold-change) fallback path

## Outputs

- meta_results.csv with per-compound meta-analysis model applied (fixed or random), combined P-value, and I2
- Compound-level significance calls, e.g. Proline significant under random-effects with I2 = 87.1%

## How to apply

Load the compound-level dataset (per-study effect sizes and standard errors for each amino acid compound) into R. For each compound, fit a fixed-effects inverse-variance meta-analysis with the 'meta' package (metagen/metabin family; metagen for generic effect sizes, metabin for binary data). Compute Cochran's Q and the I2 heterogeneity statistic per compound and switch to a DerSimonian-Laird random-effects model whenever I2 > 40%. Extract the model applied, combined P-value and I2 per compound into meta_results.csv. Validate by checking that the target compound reproduces the reported result — here Proline significant under the random-effects model with I2 = 87.1%. Note the rationale: classical pooling needs variances, which metabolomics studies often omit; when only P-values and fold-changes are reported, a weighted non-integral approach (e.g. the amanida package with weighted Fisher combination of P-values and study-size-weighted fold-change averaging) is the alternative.

## Related tools

- **meta** (R package used to fit fixed-effects inverse-variance and DerSimonian-Laird random-effects models per compound (metagen/metabin functions) and to compute Cochran's Q and I2)
- **R** (Analysis environment in which the compound-wise meta-analysis loop, threshold switching, and meta_results.csv export are scripted)
- **amanida** (Alternative R package for the same metabolomics meta-analysis goal when only significance and fold-change (non-integral data) are reported) — https://github.com/mariallr/amanida

## Examples

```
library(meta); res <- metagen(TE = es, seTE = se, studlab = study, data = aa, comb.fixed = TRUE, comb.random = TRUE)
```

## Evaluation signals

- Proline reproduces the reported significance under the random-effects model with I2 = 87.1%
- Every compound row in meta_results.csv records the model applied (fixed vs random) consistent with the I2 > 40% switching rule
- Each compound's Cochran's Q and I2 are computed and present, with I2 in the valid 0–100% range
- Combined P-values for compounds are consistent with the inverse-variance weights implied by the supplied standard errors
- Fixed-effects and random-effects estimates diverge most for the highest-heterogeneity compounds (e.g. those with I2 near 87%)

## Limitations

- Requires per-study effect sizes and standard errors; metabolomics studies rarely disclose all statistical information needed for a classical meta-analysis
- Heterogeneity is high in these datasets (Proline I2 = 87.1%), so fixed-effects results alone are not defensible; the random/fixed model choice must be reported per compound
- Identifier inconsistencies across studies (compound naming) must be curated/harmonized before pooling, or duplicates will distort vote counts and pooled effects

## Evidence

- [other] Repeating the classical meta-analysis with the 'Meta' R package yielded significant results for Proline under the random-effect model, with heterogeneity I2 = 87.1%.: [evidence span withheld — anchor status: not_found]
- [other] For each compound, fit a fixed-effects inverse-variance meta-analysis model using the 'meta' R package (metagen/metabin family of functions).: [evidence span withheld — anchor status: not_found]
- [other] Compute Cochran's Q and the I2 heterogeneity statistic for each compound and switch to a DerSimonian-Laird random-effects model whenever I2 > 40%.: [evidence span withheld — anchor status: not_found]
- [abstract] Here, we present a meta-analysis approach using only the most reported statistical parameters in this field: P-value and fold-change.: "Here, we present a meta-analysis approach using only the most reported statistical parameters in this field: P-value and fold-change."
- [abstract] metabolomics studies rarely disclose all the statistical information needed to perform a meta-analysis: "Unfortunately, metabolomics studies rarely disclose all the statistical information needed to perform a meta-analysis."
