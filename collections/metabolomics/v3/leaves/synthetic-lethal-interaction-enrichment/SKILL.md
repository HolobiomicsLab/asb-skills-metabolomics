---
name: synthetic-lethal-interaction-enrichment
description: Use when you have per-metabolite feature-importance rankings from trained
  random-forest models (e.g. Metab8D top-20 features) and want to test whether top
  predictors are enriched for curated SynLethDB synthetic-lethal pairs versus a random-feature
  null.
when_to_use_negative:
- The trained-model feature-importance output (Metab8D_network.xlsx or equivalent
  top-predictor list) is not yet available — this skill operates on model outputs,
  not raw CCLE omics tables
- The goal is validation in an in vivo or non-CCLE setting — the article warns such
  models are 'not directly transferrable' to other sources due to in vitro vs in vivo
  differences and limited feature overlap
- A causal mechanism for a single metabolite is required — the article cautions that
  'guilt by association' network associations are descriptive and not necessarily
  mechanistic
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: scikit-learn
  role: Random Forest Regressor whose feature-importance rankings supply the top-10
    predictors per metabolite that are tested for SL enrichment
- name: Python
  role: Scripting environment for the enrichment workflow (Metab8D scripts, scipy
    t-test, CSV output)
  repo: sriram-lab/Metab8D
- name: machine learning (random forest regression)
  role: Generates the per-metabolite feature importances; random forests outperformed
    Ridge, Lasso, XGBoost, and SVMs in cross-validation benchmarking
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:32d86a9e81b04c9e9c50c22b9916e9bef2ad1d009f0774a22ff0d809bf91eeb4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - machine learning (random forest regression)
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# synthetic-lethal-interaction-enrichment

## Summary

Tests whether the top machine-learning predictors of each metabolite are enriched for curated synthetic-lethal (SL) gene pairs beyond random chance, as a signature of compensatory metabolic regulation. In Recon8D it uses random-forest feature importances from Metab8D and the SynLethDB SL set.

## When to use

Use when you have per-metabolite feature-importance rankings from trained random-forest models (e.g. Metab8D top-20 features) and want to test whether top predictors are enriched for curated SynLethDB synthetic-lethal pairs versus a random-feature null.

## When NOT to use

- The trained-model feature-importance output (Metab8D_network.xlsx or equivalent top-predictor list) is not yet available — this skill operates on model outputs, not raw CCLE omics tables
- The goal is validation in an in vivo or non-CCLE setting — the article warns such models are 'not directly transferrable' to other sources due to in vitro vs in vivo differences and limited feature overlap
- A causal mechanism for a single metabolite is required — the article cautions that 'guilt by association' network associations are descriptive and not necessarily mechanistic

## Inputs

- Random-forest feature-importance rankings per metabolite per feature set (e.g. Metab8D_network.xlsx top-20 features with confidence scores 0-8)
- SynLethDB curated synthetic-lethal gene-pair list
- Feature pool (feature names) for each omics layer used to draw the random null sets
- RF_results table of Pearson's correlations and Bonferroni-corrected P values to confirm which metabolite models are significant

## Outputs

- synthetic_lethal_enrichment.csv with per-metabolite observed SL interaction counts, null-distribution statistics, enrichment fold-change, and p values

## How to apply

Load the trained random-forest feature-importance rankings from the Metab8D output files (top 20 features per metabolite across nine feature sets, excluding genomic mutations), isolating the top 10 predictors for each of the 225 metabolites in each omics-layer model. Load the SynLethDB curated synthetic-lethal gene-pair list and build a set of SL pairs. For each metabolite, count how many of its top-10 predictor pairs appear in the SynLethDB SL set to yield the observed SL interaction count per metabolite. Build a null distribution by drawing 50 random sets of 10 features per metabolite from the same feature pool and recomputing the SL overlap count for each. Compare observed to null counts per metabolite with a two-sample t-test (scipy.stats.ttest_ind) to get a fold-change and p value, then aggregate across features of the transcriptomics model, which is expected to show the strongest enrichment (52 SL interactions across 45 metabolites, 27-fold, p=1e-34). Write the per-metabolite table as synthetic_lethal_enrichment.csv.

## Related tools

- **scikit-learn** (Random Forest Regressor whose feature-importance rankings supply the top-10 predictors per metabolite that are tested for SL enrichment)
- **Python** (Scripting environment for the enrichment workflow (Metab8D scripts, scipy t-test, CSV output)) — sriram-lab/Metab8D
- **machine learning (random forest regression)** (Generates the per-metabolite feature importances; random forests outperformed Ridge, Lasso, XGBoost, and SVMs in cross-validation benchmarking)

## Examples

```
from scipy.stats import ttest_ind; stat, p = ttest_ind(observed_SL_counts, null_SL_counts)  # 50 random draws of 10 features per metabolite
```

## Evaluation signals

- The transcriptomics-model aggregate must reproduce the reported values within numerical tolerance: 52 SL interactions, 45 metabolites, 27-fold enrichment, p=1e-34
- Metabolite models fed into the enrichment should be statistically significant under the Bonferroni-corrected threshold 0.05/225 before their top features are trusted
- Each null set must be drawn from the same feature pool as the real model (50 draws of 10 features per metabolite) — a mismatched pool inflates or deflates the fold-change
- The output CSV must contain, per metabolite, the observed SL count, the null-distribution comparison (two-sample t-test), fold-change, and p value
- Confidence scores (0-8, number of controls out of 8 experiments in which a feature appeared in the top 20) can be cross-checked to confirm top features are stable, not one-fold artifacts

## Limitations

- Random-forest associations are 'guilt by association' inferences; the article notes these are descriptive and may not identify causative mechanisms, and experimental validation would require immense effort
- Genomic mutations are excluded from the random-forest feature sets because of their binary structure, so SL enrichment cannot be assessed for mutation-derived predictors
- Only the top 20 features per metabolite are retained in Metab8D_network.xlsx, and the 225-metabolite panel covers only a fraction of cellular metabolites across compartments
- Enrichment relies on the CCLE-trained models and may not transfer to other cell panels or in vivo data

## Evidence

- [results] The transcriptomics model showed the strongest enrichment of curated SynLethDB SL interactions among top predictors, with 52 interactions involving 45 metabolites: [evidence span withheld — anchor status: not_found]
