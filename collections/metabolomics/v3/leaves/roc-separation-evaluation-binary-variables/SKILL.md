---
name: roc-separation-evaluation-binary-variables
description: Use when a lipidomics table with lipid abundances and attached study
  variables contains a binary nominal variable (e.g., condition/treatment group) and
  you need to quantify which selected lipid subset separates the two classes.
when_to_use_negative:
- The study variable is continuous/numerical rather than a binary nominal variable
  — rank models by AIC only, ROC accuracy does not apply
- The imported data contains fewer than two study variables, so a 'Complete feature
  analysis' relating variables to each other cannot be computed
- The goal is lipidome structural comparison (MCS + Jaccard distances, PCA lipid space
  model, Hausdorff distances) rather than identifying lipids that distinguish study
  variables
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: LipidSpace
  role: Standalone workstation application whose built-in feature analysis module
    runs SFS-based model building and ROC accuracy evaluation for binary study variables
  repo: https://github.com/lifs-tools/lipidspace
- name: Goslin
  role: C++ library used by LipidSpace to parse and standardize lipid names before
    feature analysis
  repo: https://github.com/lifs-tools/cppgoslin
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:8e5079c3beeefa0da69494042a91f7cca8de41129b5b45ff5e83f79d70233875
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - LipidSpace
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ROC separation evaluation for binary study variables

## Summary

LipidSpace's feature analysis evaluates how well the lipid subsets selected by sequential forward selection (SFS) separate binary nominal study variables by additionally computing ROC accuracy alongside the AIC-ranked regression models, writing the per-variable results to feature_analysis_results.csv.

## When to use

Use when a lipidomics table with lipid abundances and attached study variables contains a binary nominal variable (e.g., condition/treatment group) and you need to quantify which selected lipid subset separates the two classes.

## When NOT to use

- The study variable is continuous/numerical rather than a binary nominal variable — rank models by AIC only, ROC accuracy does not apply
- The imported data contains fewer than two study variables, so a 'Complete feature analysis' relating variables to each other cannot be computed
- The goal is lipidome structural comparison (MCS + Jaccard distances, PCA lipid space model, Hausdorff distances) rather than identifying lipids that distinguish study variables

## Inputs

- Lipidome abundance table (csv, xlsx, or mzTab-M; flat or pivot layout) with lipid species as features and sample-associated study variables as targets, lipid names parsed/standardized via Goslin
- At least one binary nominal study variable (e.g., condition, treatment vs. control)
- Example dataset shipped with LipidSpace (examples/Example-Dataset.xlsx)

## Outputs

- feature_analysis_results.csv containing selected lipid subsets per study variable, per-round AIC values, and ROC accuracies for binary nominal variables
- Per-study-variable best (lowest-AIC) regression model with the lipid subset that best describes the variable

## How to apply

Load a lipidome table (csv/xlsx, flat or pivot; e.g., the bundled examples/Example-Dataset.xlsx) with lipids as features and study variables as targets. For each study variable, run heuristic sequential forward selection: fit a multiple linear regression on every candidate lipid subset, score each model by AIC, extend the best subset each round with supersets of the previously chosen features, and stop after the first √n rounds (n = number of lipids) to limit computation and avoid overfitting. For binary nominal variables, additionally evaluate the separation quality of the selected subset via ROC accuracy, while continuous variables are ranked by AIC alone. In the 'Complete feature analysis' mode, each study variable's regression/classification model is applied to all remaining study variables to compute the accuracy, a measure of the dependency between any two study variables. Record the selected lipid subsets, per-round AIC values, and ROC accuracies in feature_analysis_results.csv.

## Related tools

- **LipidSpace** (Standalone workstation application whose built-in feature analysis module runs SFS-based model building and ROC accuracy evaluation for binary study variables) — https://github.com/lifs-tools/lipidspace
- **Goslin** (C++ library used by LipidSpace to parse and standardize lipid names before feature analysis) — https://github.com/lifs-tools/cppgoslin

## Examples

```
LipidSpace menu → File → Import example dataset; then Analysis → Complete feature analysis (results in feature_analysis_results.csv)
```

## Evaluation signals

- feature_analysis_results.csv contains a ROC accuracy column populated for binary nominal study variables but not for numerical ones
- The selected subset for each variable corresponds to the model with the overall lowest AIC across the √n SFS rounds
- Number of selected lipids per variable never exceeds √n (n = total number of lipid features)
- In Complete feature analysis, each variable's model applied to all remaining study variables yields a reported accuracy value measuring the dependency between variable pairs
- Running the bundled example dataset reproduces the per-variable AIC and ROC values reported in the feature analysis tutorial

## Limitations

- SFS is explicitly a heuristic, fast approach — the selected subset is not guaranteed to be the globally optimal feature set
- Selection is terminated after only the first √n steps to limit computational time and avoid overfitting, so late-round feature combinations are never explored
- Analyses cannot currently be saved or reloaded in LipidSpace, so ROC/AIC results must be captured in feature_analysis_results.csv during the session

## Evidence

- [intro] SFS + ROC for binary variables drives feature analysis: "The built-in feature analysis module assists in determining which lipids exert the highest effect in distinguishing between individual study variables or combinations thereof"
- [methods] SFS heuristic named in methods: "We, therefore, applied an efficient, fast heuristic sequential forward selection (SFS) of features."
- [abstract] Machine-learning determination of lipid subsets for study variables: "LipidSpace can determine lipid subsets across all lipidomes that describe these study variables well by utilizing machine-learning approaches."
- [readme] Regression/classification models built per study variable to compute accuracy: "When your imported data contains at least two study variables, LipidSpace creates for each study variable a regression/classification model and applies this model on all remaining study variables to"
- [readme] Interactive tutorial covers feature analysis: "The tutorials are give an introduction to i) data import, ii) handling of the UI for result interpretation, iii) feature analysis, and iv) quality control methods."
