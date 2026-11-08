---
name: random-forest-feature-importance-extraction
description: Use when trained random forest regression models (e.g. per-metabolite
  models over CCLE multiomics features) need their feature importances ranked, top-k
  features extracted, and robustness-confidence scores computed.
when_to_use_negative:
- Extracting features from genomic mutation models — mutations were excluded from
  random forest results due to their binary structure
- Transferring the trained models or extracted features to predict metabolomics from
  other (e.g. in vivo) sources — the article states models are not directly transferable
  due to in vitro/in vivo differences and limited feature overlap
- Input omics data with too few samples/cell lines — differences in sample quantities
  influence accuracy and such omics may not perform well
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: Provides the Random Forest Regressor algorithm whose feature importance values
    are ranked and converted into the top-20/confidence-score network table
- name: Python
  role: Runs the ML_function.ipynb pipeline that trains models, assesses accuracy,
    and generates feature importances
- name: XGBoost
  role: Benchmarked against random forest regression to justify the choice of RF;
    not used for the final feature extraction
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:ebb9d9e2db3002c0b87e8bab7413fa51afbe97b671215edca524b55f159b8f79
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - SARTRE
  - XGBoost
  - machine learning (random forest regression)
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# random-forest-feature-importance-extraction

## Summary

Extracts the top-20 most important predictor features from scikit-learn Random Forest Regressor models trained per metabolite across nine feature sets of CCLE omics data, and assigns each feature a 0–8 confidence score based on how many of 8 control experiments it reappears in.

## When to use

Use when trained random forest regression models (e.g. per-metabolite models over CCLE multiomics features) need their feature importances ranked, top-k features extracted, and robustness-confidence scores computed.

## When NOT to use

- Extracting features from genomic mutation models — mutations were excluded from random forest results due to their binary structure
- Transferring the trained models or extracted features to predict metabolomics from other (e.g. in vivo) sources — the article states models are not directly transferable due to in vitro/in vivo differences and limited feature overlap
- Input omics data with too few samples/cell lines — differences in sample quantities influence accuracy and such omics may not perform well

## Inputs

- Trained random forest regression models with feature importances (one model per metabolite per feature set, from ML_function.ipynb / Recon8D Synapse page)
- CCLE multiomics feature matrices from 8 biomolecular classes (DepMap-sourced), NaN-replaced with zeros, zero-variance features removed, Z-score normalized within each CV fold
- Metabolomics panel of 225 metabolites across matched CCLE cell lines
- 8 control experiment feature-importance rankings (lung, hematopoietic/lymphoid and their complements; suspension and adherent; primary and metastatic)

## Outputs

- Metab8D_network.xlsx: top 20 features for 2,025 metabolite models (nine feature sets × 225 metabolites, excluding genomic mutations) with confidence scores 0–8
- Per-metabolite feature importance rankings (optionally supplemented by Shapley analysis for select metabolites)
- RF_results: Pearson's correlations and Bonferroni-corrected P values (0.05/225) for all metabolite models from each of nine feature sets

## How to apply

Load the trained scikit-learn Random Forest Regressor models produced by the ML_function.ipynb pipeline — trained per metabolite on ten feature sets from eight biomolecular classes (CNV, DNA methylation, histone PTMs, coding/non-coding transcripts, splicing, proteomics, phosphoproteomics; genomic mutations are excluded due to their binary structure) over ~1000 matched CCLE cell lines. Rank features by importance and extract the top 20 per metabolite per feature set. Repeat the top-20 extraction across the 8 control experiments (lung, hematopoietic/lymphoid and complements; suspension and adherent; primary and metastatic) and tally, for each feature–metabolite pair, in how many controls it appears in the top 20 to yield a confidence score on the 0–8 scale. Assemble the pairs and scores into a network table (Metab8D_network.xlsx). Interpret high-confidence features with no previously identified mechanistic relationship as priorities for further study.

## Related tools

- **scikit-learn** (Provides the Random Forest Regressor algorithm whose feature importance values are ranked and converted into the top-20/confidence-score network table)
- **Python** (Runs the ML_function.ipynb pipeline that trains models, assesses accuracy, and generates feature importances)
- **XGBoost** (Benchmarked against random forest regression to justify the choice of RF; not used for the final feature extraction)

## Examples

```
jupyter nbconvert --to notebook --execute ML_function.ipynb
```

## Evaluation signals

- Metab8D_network.xlsx exists, is a valid Excel table, and contains top-20 features for all 2,025 trained metabolite models (nine feature sets for 225 metabolites)
- All confidence scores fall within the 0–8 range and match the number of controls (out of 8 experiments) in which each feature appeared in the top 20
- Genomic mutation feature sets are absent from the network output (excluded from RF results)
- Model validity is supported by Pearson's correlation between predicted and true metabolite concentrations with significance at Bonferroni-corrected P value 0.05/225
- Downstream checks: transcriptomics top-10 predictors should reproduce 52 SynLethDB SL interactions across 45 metabolites (27-fold, p=1e-34)

## Limitations

- The 'guilt by association' strategy of network inference may not identify causative mechanisms; associations are descriptive, not necessarily mechanistic
- Analysis was done in cancer cell lines in culture and may not reflect metabolism in vivo; models are not directly transferable to other sources
- The 225-metabolite panel and 90 targeted phosphoproteins cover only a fraction of cellular metabolites and the full phosphoprotein array
- Experimentally validating even a small percentage of proposed interactions would require an immense effort

## Evidence

- [discussion] Random forests were trained using Sci-kit Learn's Random Forest Regressor algorithm: [evidence span withheld — anchor status: not_found]
