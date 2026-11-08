---
name: control-experiment-cross-condition-feature-tallying
description: Use when random forest regression feature importances have been generated
  per metabolite per feature set and you must separate robust regulator–metabolite
  associations from confounders driven by tissue type, culture conditions (suspension/adherent),
  or sample origin (primary/metastatic).
when_to_use_negative:
- When the goal is single-omics model performance or accuracy assessment — use RF_results
  (Pearson's correlations and Bonferroni-corrected P values) instead of the confidence
  tally
- When the input omics are not cell-line (CCLE/NCI60) feature tables, since models
  are not directly transferrable to other sources due to in vitro vs in vivo differences
- When causal mechanistic relationships are required — the confidence score reflects
  descriptive 'guilt by association' predictability, not causation
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2269
tools:
- name: scikit-learn Random Forest Regressor
  role: Source of the ranked feature importances whose top-20 membership is tallied
    across control experiments
  repo: https://github.com/sriram-lab/Metab8D
- name: Python
  role: Language of the ML_function.ipynb pipeline that trains models, extracts top-20
    features, and assembles the network table
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:ca6425ee8af889dacd56d00e918a265e376c0b3a30d35dec98644ffe4222215f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - scikit-learn
  - scikit-learn Random Forest Regressor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# control-experiment-cross-condition-feature-tallying

## Summary

Assigns a 0–8 confidence score to each feature–metabolite pair by tallying how many of 8 control experiments (cross-tissue, culture-mode, and origin conditions) the feature appears in the top-20 random forest feature importances, producing a network table (Metab8_network.xlsx).

## When to use

Use when random forest regression feature importances have been generated per metabolite per feature set and you must separate robust regulator–metabolite associations from confounders driven by tissue type, culture conditions (suspension/adherent), or sample origin (primary/metastatic).

## When NOT to use

- When the goal is single-omics model performance or accuracy assessment — use RF_results (Pearson's correlations and Bonferroni-corrected P values) instead of the confidence tally
- When the input omics are not cell-line (CCLE/NCI60) feature tables, since models are not directly transferrable to other sources due to in vitro vs in vivo differences
- When causal mechanistic relationships are required — the confidence score reflects descriptive 'guilt by association' predictability, not causation

## Inputs

- Trained random forest regression models and their feature importances (from ML_function.ipynb, scikit-learn Random Forest Regressor) for each metabolite and each feature set
- CCLE-derived feature matrices for each control condition (lung, hematopoietic/lymphoid and complements; suspension and adherent; primary and metastatic cell lines)
- List of 225 metabolites and 9 feature sets (excluding genomic mutations)

## Outputs

- Metab8_network.xlsx: top 20 features for 2,025 trained metabolite models (nine feature sets × 225 metabolites) with confidence scores (0–8)
- Per-feature–metabolite confidence tally across the 8 control experiments

## How to apply

Load the trained scikit-learn Random Forest Regressor models produced by the ML_function.ipynb pipeline for each of the 225 metabolites and each individual feature set, rank features by importance, and extract the top 20 per metabolite per set. Repeat the top-20 extraction across the 8 control experiments spanning lung vs hematopoietic/lymphoid tissue and their complements, suspension vs adherent culture, and primary vs metastatic origin. For each feature–metabolite pair, tally the number of controls in which the feature appears in the top 20, yielding a confidence score on a 0–8 scale where higher scores indicate the association is not explained by a single condition. Assemble pairs with their confidence scores into Metab8_network.xlsx. The rationale is that features persisting across all 8 control conditions are unlikely to reflect confounders such as tissue type or culture mode.

## Related tools

- **scikit-learn Random Forest Regressor** (Source of the ranked feature importances whose top-20 membership is tallied across control experiments) — https://github.com/sriram-lab/Metab8D
- **Python** (Language of the ML_function.ipynb pipeline that trains models, extracts top-20 features, and assembles the network table) — https://github.com/sriram-lab/Metab8D

## Examples

```
pip install -r requirements.txt; jupyter notebook ML_function.ipynb
```

## Evaluation signals

- Verify Metab8_network.xlsx exists and is a valid Excel table
- All confidence scores in the table fall within the 0–8 range
- Row set matches the Metab8D network deposit (top 20 features × 2,025 models: 9 feature sets × 225 metabolites)
- Tallies are consistent with re-running top-20 extraction across exactly the 8 control conditions (lung/hematopoietic and complements; suspension/adherent; primary/metastatic)
- Genomic mutations are excluded from tallied feature sets (excluded due to binary structure)

## Limitations

- Mutations were excluded from random forest model results due to their binary structure
- The 'guilt by association' network inference strategy may not identify causative mechanisms; associations from systems-level data are descriptive, not necessarily mechanistic
- Analysis was done in cancer cell lines in culture and may not reflect metabolism in vivo; models are not directly transferrable to other sources due to in vitro/in vivo differences, cell line panel differences, and limited feature overlap
- Experimentally validating the proposed interactions, even for a small percentage, would require an immense effort

## Evidence

- [results] A confidence score (0-8) was assigned to each feature based on the number of the 8 control experiments in which that feature appeared among the top 20 features.: [evidence span withheld — anchor status: not_found]
- [discussion] Random forests were trained using Sci-kit Learn's Random Forest Regressor algorithm with the following parameters: [evidence span withheld — anchor status: not_found]
- [discussion] the 'guilt by association' strategy used by network inference methods may not identify causative mechanisms: [evidence span withheld — anchor status: not_found]
