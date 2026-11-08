---
name: probabilistic-quotient-normalization
description: Use when a metabolomics feature table shows unwanted sample-to-sample
  variation (e.g. dilution or total-signal differences) and a normalization robust
  to a few dominant metabolite changes is needed before statistical analysis.
when_to_use_negative:
- The feature table has not yet been imputed / cleaned (PQN should be applied during
  preprocessing on a completed feature table)
- A normalization tied to measured concentration is required, e.g. when internal standard
  signals measured in the same sample are available and preferred
- A distribution-shape correction across samples is the goal rather than a global
  per-sample scaling factor — quantile normalization, which forces identical empirical
  distributions, is the better choice
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: MetaboReport
  role: R Shiny app whose data preprocessing module implements PQN (and four other
    normalization methods) to reduce sample-to-sample variation in metabolomics feature
    tables
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Language in which MetaboReport (and therefore the PQN workflow) is installed
    and run, e.g. via remotes::install_github and run_app()
  repo: https://github.com/YonghuiDong/MetaboReport
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:3c16af7665129800f3695331243d24020a971dae61653a230321f95f529fdd5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# probabilistic-quotient-normalization

## Summary

Probabilistic quotient normalization (PQN) is one of the five sample-to-sample normalization methods implemented in MetaboReport for metabolomics feature tables; each sample is scaled by the median of its feature-wise quotient ratios relative to a reference sample, making it robust to large concentration changes in a few metabolites.

## When to use

Use when a metabolomics feature table shows unwanted sample-to-sample variation (e.g. dilution or total-signal differences) and a normalization robust to a few dominant metabolite changes is needed before statistical analysis.

## When NOT to use

- The feature table has not yet been imputed / cleaned (PQN should be applied during preprocessing on a completed feature table)
- A normalization tied to measured concentration is required, e.g. when internal standard signals measured in the same sample are available and preferred
- A distribution-shape correction across samples is the goal rather than a global per-sample scaling factor — quantile normalization, which forces identical empirical distributions, is the better choice

## Inputs

- Imputed metabolomics feature table (samples x features), e.g. the cancerCell demo dataset shipped with MetaboReport
- Reference sample or reference intensity vector for quotient computation

## Outputs

- PQN-normalized feature table preserving the original samples x features dimensions
- Per-sample median quotient scaling factors (implicit intermediate)

## How to apply

Load the feature table (e.g. the imputed cancerCell demo table shipped with the MetaboReport R package) as a samples-by-features matrix. Choose a reference sample (or a reference vector such as the median of all samples) and compute, for every feature and every sample, the quotient of that sample's feature intensity divided by the reference value. For each sample, take the median of these feature-wise quotients to obtain a per-sample scaling factor, then divide all of that sample's feature intensities by this median quotient. This targets the median ratio rather than the mean, so total-ion drift and dilution effects are corrected while spike-like changes in a minority of metabolites do not dominate the scaling factor. MetaboReport implements PQN alongside internal standard-based normalization, linear baseline normalization (mean/median), and quantile normalization, so PQN can be applied and compared with the other four methods on the same imputed input table.

## Related tools

- **MetaboReport** (R Shiny app whose data preprocessing module implements PQN (and four other normalization methods) to reduce sample-to-sample variation in metabolomics feature tables) — https://github.com/YonghuiDong/MetaboReport
- **R** (Language in which MetaboReport (and therefore the PQN workflow) is installed and run, e.g. via remotes::install_github and run_app()) — https://github.com/YonghuiDong/MetaboReport

## Examples

```
remotes::install_github("YonghuiDong/MetaboReport"); MetaboReport::run_app()
```

## Evaluation signals

- The normalized table has exactly the same dimensions as the input feature table (same samples and same features)
- Each sample's normalized values equal its original values divided by the sample's median feature-wise quotient relative to the reference sample
- Sample-to-sample variation (e.g. total signal spread across samples) is visibly reduced relative to the imputed input table
- If run alongside the other four MetaboReport normalization methods, each method's exported CSV exists and preserves input dimensions
- Quotient ratios computed on the normalized table are centered around 1 (median quotient per sample ≈ 1 relative to the reference)

## Limitations

- PQN is one of five alternative normalizations in MetaboReport; the article does not prescribe which method is optimal, so the choice must be validated per dataset
- Scaling depends on the chosen reference sample/vector; a poor reference can bias the median quotient
- Analysis helpers in MetaboReport are internal and can evolve without a compatibility contract, so the exact PQN implementation may change between releases

## Evidence

- [abstract] five normalization methods: [evidence span withheld — anchor status: not_found]
- [other] PQN workflow detail: [evidence span withheld — anchor status: not_found]
- [methods] cancerCell dataset: "- `cancerCell (MetaboReport)`"
- [readme] app scope: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [readme] API stability caveat: "Analysis helpers are internal so their implementation can evolve without creating an accidental compatibility contract."
