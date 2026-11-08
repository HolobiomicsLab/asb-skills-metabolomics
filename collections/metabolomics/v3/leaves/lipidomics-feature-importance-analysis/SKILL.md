---
name: lipidomics-feature-importance-analysis
description: Use when a lipidomics abundance table (csv/xlsx, flat or pivot, or mzTab-M)
  with attached study variables must be reduced to the lipid subset that explains
  each variable.
when_to_use_negative:
- The goal is structural comparison, lipidome similarity, dendrogram/clustering, or
  quality control — those are separate LipidSpace modules (MCS/Jaccard distances,
  QC workflow), not feature analysis.
- The imported data contains no study variables attached to the samples — the regression/classification
  models are built per study variable, so SFS has no targets to explain.
- Only a single study variable is present and a full cross-variable dependency analysis
  is desired — LipidSpace's 'Complete feature analysis' requires at least two study
  variables.
tools:
- name: LipidSpace
  role: Standalone workstation application (C++/Qt) whose built-in feature analysis
    module runs the SFS regression/classification models and writes feature_analysis_results.csv
  repo: https://github.com/lifs-tools/lipidspace
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:c3ca4b0e85d4ca883858fc95555e07c610976435bfdc3f5d6628fa105476f165
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - LipidSpace
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# lipidomics-feature-importance-analysis

## Summary

A feature analysis and selection method from LipidSpace that identifies, per study variable, the subset of lipids that best describes it via heuristic sequential forward selection (SFS) scored by AIC, with ROC accuracy for binary nominal variables.

## When to use

Use when a lipidomics abundance table (csv/xlsx, flat or pivot, or mzTab-M) with attached study variables must be reduced to the lipid subset that best explains each variable.

## When NOT to use

- The goal is structural comparison, lipidome similarity, dendrogram/clustering, or quality control — those are separate LipidSpace modules (MCS/Jaccard distances, QC workflow), not feature analysis.
- The imported data contains no study variables attached to the samples — the regression/classification models are built per study variable, so SFS has no targets to explain.
- Only a single study variable is present and a full cross-variable dependency analysis is desired — LipidSpace's 'Complete feature analysis' requires at least two study variables.

## Inputs

- Lipidome abundance table in csv or xlsx (flat or pivot layout: lipids row-wise/samples column-wise or vice versa)
- Lipidomics data in mzTab-M format
- Sample-associated study variables (categorical or numerical, e.g., body mass index, tissue, condition)

## Outputs

- Selected lipid subsets, one per study variable
- Per-round AIC values of the sequential forward selection models
- ROC accuracy values for binary nominal variables
- feature_analysis_results.csv summarizing the above

## How to apply

Load a lipidome table (e.g., LipidSpace's 'examples/Example-Dataset.xlsx' via File → Import example dataset) treating lipids as features and categorical/numerical study variables as targets. For each study variable, run heuristic sequential forward selection: start with zero features, fit a multiple linear regression for every single lipid, and score each model by AIC. In each subsequent round, extend the current subset by testing all supersets containing the previously chosen feature(s) and keep the model with the lowest AIC. Terminate after only the first √n steps (n = number of lipids) to limit computational time and avoid overfitting. Pick per study variable the model with the overall best (lowest) AIC; for binary nominal variables additionally evaluate separation quality via ROC accuracy, and write selected subsets, per-round AIC values, and ROC accuracies to feature_analysis_results.csv.

## Related tools

- **LipidSpace** (Standalone workstation application (C++/Qt) whose built-in feature analysis module runs the SFS regression/classification models and writes feature_analysis_results.csv) — https://github.com/lifs-tools/lipidspace

## Evaluation signals

- feature_analysis_results.csv is produced containing, per study variable, the selected lipid subset, per-round AIC values, and ROC accuracies where applicable.
- The number of selected features per study variable does not exceed √n, where n is the total number of lipids/features in the table.
- The best AIC reported per variable corresponds to the lowest AIC across all SFS rounds (AIC of the chosen model is ≤ AIC of every intermediate model).
- Binary nominal study variables carry an accompanying ROC accuracy; continuous variables carry regression AIC only.
- Re-running the analysis on the bundled Example-Dataset.xlsx yields consistent selected lipid subsets for the same study variables.

## Limitations

- SFS is a greedy heuristic; it does not guarantee the globally optimal lipid subset, only the best found within the first √n rounds.
- Selection is terminated after √n steps to limit computational time and avoid overfitting, so larger discriminative lipid combinations beyond √n features are never evaluated.
- LipidSpace analyses cannot currently be stored or loaded ('At the moment it is not possible to store or load an analysis.'), so the SFS results must be captured via the exported results file.

## Evidence

- [methods] SFS heuristic scoring: "We, therefore, applied an efficient, fast heuristic sequential forward selection (SFS) of features."
- [intro] Purpose of feature analysis module: "The built-in feature analysis module assists in determining which lipids exert the highest effect in distinguishing between individual study variables"
- [abstract] Machine-learning subsets per study variable: "LipidSpace can determine lipid subsets across all lipidomes that describe these study variables well by utilizing machine-learning approaches."
- [readme] Per-variable regression/classification models: "When your imported data contains at least two study variables, LipidSpace creates for each study variable a regression/classification model"
- [readme] Feature analysis covered by built-in tutorial: "The tutorials are give an introduction to i) data import, ii) handling of the UI for result interpretation, iii) feature analysis"
