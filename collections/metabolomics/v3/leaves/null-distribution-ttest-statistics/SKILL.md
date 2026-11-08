---
name: null-distribution-ttest-statistics
description: Use when an observed enrichment statistic (e.g., synthetic-lethal pair
  overlaps among top-10 metabolite predictors) must be compared against a random null
  distribution built from the same feature pool.
when_to_use_negative:
- The goal is causal mechanism discovery — the 'guilt by association' enrichment is
  descriptive and does not identify causative mechanisms
- The trained CCLE cell-line models would be applied to in vivo or other external
  metabolomics sources — the article states such models are not directly transferrable
- No curated SL reference (e.g., SynLethDB) is available for the gene pairs in the
  feature pool
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: Python (scipy.stats.ttest_ind)
  role: Runs the two-sample t-test comparing observed SL counts to the per-metabolite
    null distribution and computes fold-change
  repo: sriram-lab/Metab8D
- name: scikit-learn
  role: Provides the Random Forest Regressor feature importances that define the top-10
    predictor lists fed into the enrichment test
- name: Random forest regression
  role: Generates the metabolite-level feature-importance rankings whose top predictors
    are tested for SL enrichment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:0d446654cfc8113414aff674d4db1ef074746aed79f2d4c07c5e0c870c73070c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - Python (scipy.stats.ttest_ind)
  - Random forest regression
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# null-distribution-ttest-statistics

## Summary

Tests whether observed overlap counts (e.g., SynLethDB synthetic-lethal pairs among top metabolite predictors from the Recon8D/Metab8D random-forest models) exceed a matched random null, using a two-sample t-test between observed and null counts.

## When to use

Use when an observed enrichment statistic (e.g., synthetic-lethal pair overlaps among top-10 metabolite predictors) must be compared against a random null distribution built from the same feature pool.

## When NOT to use

- The goal is causal mechanism discovery — the 'guilt by association' enrichment is descriptive and does not identify causative mechanisms
- The trained CCLE cell-line models would be applied to in vivo or other external metabolomics sources — the article states such models are not directly transferrable
- No curated SL reference (e.g., SynLethDB) is available for the gene pairs in the feature pool

## Inputs

- Random-forest feature-importance rankings per metabolite (top 20 features from Metab8D_network.xlsx / RF_results, nine feature sets for 225 metabolites)
- SynLethDB curated synthetic-lethal gene pair list
- Feature pool (omics features used per omics-layer model)

## Outputs

- Per-metabolite observed SL overlap counts and null distribution counts
- Per-metabolite enrichment fold-change and two-sample t-test p value
- synthetic_lethal_enrichment.csv summary table

## How to apply

Load the trained random-forest feature-importance rankings from the Metab8D outputs (Metab8D_network.xlsx / RF_results) and isolate the top 10 predictors for each of the 225 metabolites in each omics-layer model. Build a set of curated SL gene pairs from SynLethDB and, per metabolite, count observed predictor pairs present in the SL set. Generate 50 random draws of 10 features per metabolite from the same feature pool and compute SL overlap counts for each draw to build the null distribution. Compare observed vs. null counts per metabolite with scipy.stats.ttest_ind to get a fold-change and p value; for the transcriptomics model the expected result is 52 SL interactions across 45 metabolites, 27-fold enrichment, p=1e-34, written as synthetic_lethal_enrichment.csv.

## Related tools

- **Python (scipy.stats.ttest_ind)** (Runs the two-sample t-test comparing observed SL counts to the per-metabolite null distribution and computes fold-change) — sriram-lab/Metab8D
- **scikit-learn** (Provides the Random Forest Regressor feature importances that define the top-10 predictor lists fed into the enrichment test)
- **Random forest regression** (Generates the metabolite-level feature-importance rankings whose top predictors are tested for SL enrichment)

## Examples

```
from scipy.stats import ttest_ind; t, p = ttest_ind(observed_SL_counts, null_SL_counts, axis=1)
```

## Evaluation signals

- Reproduced transcriptomics-model statistics match the reported values (52 SL interactions, 45 metabolites, 27-fold enrichment, p=1e-34) within numerical tolerance
- Null distributions are built from 50 random draws of 10 features per metabolite drawn from the same feature pool as the observed predictors
- Output table synthetic_lethal_enrichment.csv contains per-metabolite observed counts, null counts, fold-change, and t-test p values
- Observed overlap counts per metabolite are integers bounded by the number of top-10 predictor pairs

## Limitations

- The enrichment is computed from CCLE cancer cell line models in culture and may not reflect metabolism in vivo
- Network associations from systems-level data are descriptive, and findings are not necessarily mechanistic
- Experimentally validating proposed interactions, even for a small percentage, would require an immense effort

## Evidence
