---
name: multi-feature-set-model-aggregation
description: Use when multiple random forest models (one per metabolite per feature
  set) have been trained and their feature importances must be ranked, tallied across
  control runs, and merged into a network table with confidence scores.
when_to_use_negative:
- You need causal/mechanistic relationships — the 'guilt by association' strategy
  used by network inference methods may not identify causative mechanisms
- Input is already an aggregated network table or only a single feature set without
  the 8 control experiments, so no 0–8 confidence tally can be computed
- You want to transfer the trained models to predict metabolomics from in vivo or
  other non-CCLE sources — models are not directly transferrable due to in vitro vs
  in vivo differences and limited feature overlap
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn
  role: Provides the Random Forest Regressor used to train each metabolite model and
    generate the feature importances that are aggregated into the network table
  repo: https://github.com/sriram-lab/Metab8D
- name: Python
  role: Hosts the ML_function.ipynb workflow that trains models, computes importances,
    tallies top-20 appearances across the 8 controls, and writes Metab8D_network.xlsx
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:09521efca251ae06965f23f68d21da3ffcbff016a5a73414c53f79b846df0429
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# multi-feature-set-model-aggregation

## Summary

Aggregates per-metabolite, per-feature-set random forest regression models into a single regulome network table (Metab8D_network.xlsx) by extracting each model's top 20 features and assigning a 0–8 confidence score based on how many of 8 control experiments a feature appears in.

## When to use

Use when multiple random forest models (one per metabolite per feature set) have been trained and their feature importances must be ranked, tallied across control runs, and merged into a network table with confidence scores.

## When NOT to use

- You need causal/mechanistic relationships — the 'guilt by association' strategy used by network inference methods may not identify causative mechanisms
- Input is already an aggregated network table or only a single feature set without the 8 control experiments, so no 0–8 confidence tally can be computed
- You want to transfer the trained models to predict metabolomics from in vivo or other non-CCLE sources — models are not directly transferrable due to in vitro vs in vivo differences and limited feature overlap

## Inputs

- Trained random forest regression models (scikit-learn Random Forest Regressor) per metabolite per feature set
- Feature importance rankings from ML_function.ipynb
- Top-20 feature lists from the 8 control experiments (lung, hematopoietic/lymphoid, complements, suspension/adherent, primary/metastatic)
- RF_results: Pearson's correlations and Bonferroni-corrected P values for all metabolite models from each of nine feature sets

## Outputs

- Metab8D_network.xlsx: top 20 features for 225 RF metabolite models from nine feature sets with 0–8 confidence scores
- Feature–metabolite network edge table (feature, metabolite, confidence score)

## How to apply

Load the trained scikit-learn Random Forest Regressor models produced by the ML_function.ipynb pipeline, which were each trained on one of nine feature sets (CNV, DNA methylation, histone PTMs, coding transcripts, RNA splicing, miRNA, lncRNA, proteomics, phosphoproteomics; genomic mutations excluded due to their binary structure) for each of 225 metabolites. Rank features by importance and extract the top 20 per metabolite per feature set. Re-run the top-20 extraction across the 8 control experiments (lung, hematopoietic/lymphoid, and their complements; suspension and adherent; primary and metastatic) and tally, for each feature–metabolite pair, in how many controls the feature appears in the top 20 — this count is the confidence score on the 0–8 scale. Assemble all feature–metabolite pairs with confidence scores into the network table and save as Metab8D_network.xlsx. Rationale: the extent of predictability and repeated appearance of a feature across controls is treated as indicative of the strength of its relationship with the metabolome, while controlling for confounders like tissue type and sample origin.

## Related tools

- **scikit-learn** (Provides the Random Forest Regressor used to train each metabolite model and generate the feature importances that are aggregated into the network table) — https://github.com/sriram-lab/Metab8D
- **Python** (Hosts the ML_function.ipynb workflow that trains models, computes importances, tallies top-20 appearances across the 8 controls, and writes Metab8D_network.xlsx) — https://github.com/sriram-lab/Metab8D

## Examples

```
from sklearn.ensemble import RandomForestRegressor; rf = RandomForestRegressor(); rf.fit(X_train, y_train); top20 = rf.feature_importances_.argsort()[-20:]
```

## Evaluation signals

- Metab8D_network.xlsx exists, parses as a valid Excel table, and contains entries for all 2,025 trained metabolite models (nine feature sets × 225 metabolites)
- Every confidence score in the output falls within the 0–8 range
- Each metabolite model contributes exactly its top 20 features by importance
- Genomic mutation features are absent from the aggregated network (excluded due to binary structure)
- Each feature–metabolite pair's confidence score matches the number of the 8 control experiments in which it appeared in the top 20

## Limitations

- The 'guilt by association' strategy used by network inference methods may not identify causative mechanisms; associations are descriptive and not necessarily mechanistic
- Features with high confidence scores and no previously identified mechanistic relationship should be prioritized for further study rather than treated as validated mechanisms
- Experimentally validating proposed interactions, even for a small percentage, would require an immense effort

## Evidence
