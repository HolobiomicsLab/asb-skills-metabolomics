---
name: metabolite-predictor-overlap-analysis
description: Use when you have trained per-metabolite random-forest models with ranked
  feature importances and want to quantify whether top predictors are enriched for
  curated SL gene pairs.
when_to_use_negative:
- Input is a raw omics matrix rather than ranked model feature importances — use the
  Metab8D random-forest training skill first
- The goal is validating mechanistic relationships in independent data (NCI60) rather
  than quantifying SL enrichment among predictors
- The metabolite panel is not the 225-metabolite CCLE panel, or the feature pool differs
  from the nine Metab8D feature sets
edam_operation: http://edamontology.org/operation_3452
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3524
tools:
- name: scikit-learn
  role: Trains the random-forest regressors whose feature-importance rankings supply
    the top-10 predictor sets used in the SL overlap analysis
  repo: https://github.com/sriram-lab/Metab8D
- name: Python
  role: Scripting environment for building the SL overlap counts, generating the 50-draw
    random null, and running the t-test
  repo: https://github.com/sriram-lab/Metab8D
- name: machine learning (random forest regression)
  role: Produces per-metabolite predictor rankings that feed the SynLethDB overlap
    enrichment test
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:e8fb655b62403909273278e3d6b009bfdb263eec50c1ec48a3ce84103596cc72
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - machine learning (random forest regression)
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3452
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metabolite-predictor-overlap-analysis

## Summary

Tests whether the top machine-learning predictors of metabolite levels are enriched for curated synthetic-lethal (SL) gene interactions from SynLethDB beyond random chance, as a signature of compensatory metabolic regulation in Recon8D.

## When to use

Use when you have trained per-metabolite random-forest models with ranked feature importances and want to quantify whether top predictors are enriched for curated SL gene pairs.

## When NOT to use

- Input is a raw omics matrix rather than ranked model feature importances — use the Metab8D random-forest training skill first
- The goal is validating mechanistic relationships in independent data (NCI60) rather than quantifying SL enrichment among predictors
- The metabolite panel is not the 225-metabolite CCLE panel, or the feature pool differs from the nine Metab8D feature sets

## Inputs

- Trained random-forest feature-importance rankings per metabolite (Metab8D output files on the Recon8D Synapse page)
- SynLethDB curated synthetic-lethal gene pair list
- Feature-importance table Metab8D_network.xlsx (top 20 features per metabolite per feature set with confidence scores)

## Outputs

- Per-metabolite observed SL overlap counts against a 50-draw random null
- Enrichment fold-change and two-sample t-test p value per metabolite
- Aggregated summary CSV synthetic_lethal_enrichment.csv

## How to apply

First load the trained random-forest model feature-importance rankings from the Metab8D output files (e.g. Metab8D_network.xlsx or the RF_results correlations) and isolate the top 10 predictors for each of the 225 metabolites in each omics-layer model. Second, load the SynLethDB curated synthetic-lethal gene pair list and build a set of SL gene pairs. Third, for each metabolite, count the number of pairs among its top-10 predictors that appear in the SynLethDB SL set, yielding the observed SL interaction count. Fourth, generate 50 random draws of 10 features per metabolite from the same feature pool and compute the SL overlap count for each random set to build a null distribution. Fifth, compare observed SL counts to the null distribution per metabolite using a two-sample t-test (scipy.stats.ttest_ind) to obtain an enrichment fold-change and p value. Finally, aggregate per-metabolite results across the transcriptomics-model features to reproduce the reported summary of 52 SL interactions across 45 metabolites, 27-fold enrichment, and p = 1e-34, writing the table as synthetic_lethal_enrichment.csv.

## Related tools

- **scikit-learn** (Trains the random-forest regressors whose feature-importance rankings supply the top-10 predictor sets used in the SL overlap analysis) — https://github.com/sriram-lab/Metab8D
- **Python** (Scripting environment for building the SL overlap counts, generating the 50-draw random null, and running the t-test) — https://github.com/sriram-lab/Metab8D
- **machine learning (random forest regression)** (Produces per-metabolite predictor rankings that feed the SynLethDB overlap enrichment test) — https://github.com/sriram-lab/Metab8D

## Examples

```
python ml_function.py --omics transcriptomics --top_n 10 --random_draws 50 --synlethdb synlethdb_pairs.csv --output synthetic_lethal_enrichment.csv
```

## Evaluation signals

- Reproduced transcriptomics enrichment must match the reported 52 SL interactions involving 45 metabolites, 27-fold higher than random chance, and p = 1e-34
- Each metabolite's observed SL count must come from exactly 10 top predictors, with the null built from 50 random draws of 10 features from the same feature pool
- The t-test p value must be computed with scipy.stats.ttest_ind comparing observed to null counts
- Output CSV synthetic_lethal_enrichment.csv must contain one row per metabolite with fold-change and p value
- Results should be restricted to the transcriptomics-layer model features as reported

## Limitations

- Discovery of SL interactions among top metabolite predictors supports combination-therapy design targeting compensatory mechanisms, but the associations are 'guilt by association' network inferences and not necessarily causative or mechanistic
- Analysis was done in ~1000 cancer cell lines in culture and may not reflect metabolism in vivo; models are not directly transferrable to other sources due to differences between in vitro and in vivo conditions
- The 225-metabolite panel is only a fraction of metabolites in a cell across distinct cellular compartments, so enrichment is assessed on a limited set

## Evidence

- [results] transcriptomics model showed the strongest enrichment of curated SynLethDB SL interactions among top predictors, with 52 interactions involving 45 metabolites — 27-fold higher than random chance (p-value = 1 x 10^-34): [evidence span withheld — anchor status: not_found]
- [intro] Random forest regression ML algorithms were used to generate metabolomic predictions, as they generally outperformed other ML methods based on cross-validation: [evidence span withheld — anchor status: not_found]
- [methods] Random forests were trained using Sci-kit Learn's Random Forest Regressor algorithm with the following parameters: [evidence span withheld — anchor status: not_found]
