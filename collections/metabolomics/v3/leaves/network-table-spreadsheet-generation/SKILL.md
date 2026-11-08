---
name: network-table-spreadsheet-generation
description: Use when a trained per-metabolite model set (e.g. scikit-learn Random
  Forest Regressor models over nine feature sets) must be converted into a consolidated
  network spreadsheet with control-based confidence scores.
when_to_use_negative:
- Mutations are excluded from random forest model results due to their binary structure,
  so mutation-derived features should not be tabulated into the network
- The trained models are not directly transferrable to predicting metabolomics from
  other sources due to differences between in vitro and in vivo conditions, so confidence
  scores should not be applied to out-of-panel datasets
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3360
tools:
- name: scikit-learn
  role: Random Forest Regressor whose feature importances supply the top-20 rankings
    tallied into confidence scores
- name: Python
  role: Implementation language for the ML_function.ipynb pipeline that generates
    the model importances and assembles the network table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:76e91ed5e972b0f5e8ffd97d8d9f388357b9a7f3ff0f2c9663b2238b329a354d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# network-table-spreadsheet-generation

## Summary

Assembles top-ranked ML feature–metabolite pairs into an Excel network table (Metab8_network.xlsx) where each pair carries a 0–8 confidence score counting how many of the 8 control experiments the feature appeared in for that metabolite.

## When to use

Use when a trained per-metabolite model set (e.g. scikit-learn Random Forest Regressor models over nine feature sets) must be converted into a consolidated network spreadsheet with control-based confidence scores.

## When NOT to use

- Mutations are excluded from random forest model results due to their binary structure, so mutation-derived features should not be tabulated into the network
- The trained models are not directly transferrable to predicting metabolomics from other sources due to differences between in vitro and in vivo conditions, so confidence scores should not be applied to out-of-panel datasets

## Inputs

- Trained scikit-learn Random Forest Regressor models and their feature importances from the ML_function.ipynb pipeline, one model per metabolite per feature set
- Feature importance rankings from the 8 control experiments (lung, hematopoietic/lymphoid, and their complements; suspension and adherent; primary and metastatic)

## Outputs

- Metab8_network.xlsx — Excel network table of top 20 features for all 2,025 trained metabolite models (nine feature sets for 225 metabolites) with 0–8 confidence scores

## How to apply

Load the trained random forest regression models and their feature importances produced by the ML_function.ipynb pipeline for each metabolite and each individual feature set (nine feature sets across 225 metabolites; genomic mutations are excluded). Rank features by importance and extract the top 20 per metabolite per feature set. Repeat this top-20 extraction across the 8 control experiments (lung and hematopoietic/lymphoid cell line panels and their complements; suspension and adherent; primary and metastatic), and tally for each feature–metabolite pair the number of control experiments in which it appears in the top 20 — this tally is the confidence score on the 0–8 scale. Assemble the feature–metabolite pairs with confidence scores into a network table saved as Metab8_network.xlsx, matching the Metab8D network deposit.

## Related tools

- **scikit-learn** (Random Forest Regressor whose feature importances supply the top-20 rankings tallied into confidence scores)
- **Python** (Implementation language for the ML_function.ipynb pipeline that generates the model importances and assembles the network table)

## Examples

```
python ML_function.ipynb  # train RF models per metabolite/feature set, tally top-20 appearances across 8 controls, and write Metab8_network.xlsx
```

## Evaluation signals

- Metab8_network.xlsx exists and is a valid Excel table
- All confidence scores fall within the 0–8 range
- The table covers 2,025 trained metabolite models (nine feature sets for 225 metabolites)
- The assembled table matches the Metab8D network deposit
- Each feature–metabolite confidence score equals the count of control experiments (out of 8) in which that feature appeared in the top 20 most important features

## Limitations

- Mutations were excluded from random forest model results due to their binary structure
- The 'guilt by association' strategy used by network inference methods may not identify causative mechanisms
- Models are not directly transferrable to predicting metabolomics from other sources due to differences between in vitro and in vivo conditions, cell line panels, and limited feature overlap

## Evidence

- [methods] Confidence score definition (0–8 scale): [evidence span withheld — anchor status: not_found]
- [discussion] Model training algorithm: [evidence span withheld — anchor status: not_found]
