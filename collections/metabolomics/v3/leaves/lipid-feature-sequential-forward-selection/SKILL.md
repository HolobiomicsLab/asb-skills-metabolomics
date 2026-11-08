---
name: lipid-feature-sequential-forward-selection
description: Use when a lipidomics abundance table is annotated with sample-associated
  study variables (e.g., BMI, tissue, condition) and you need the lipid subset that
  distinguishes each variable.
when_to_use_negative:
- The dataset carries no sample-associated study variables (lipid abundances only)
  — there is nothing for the regression/classification models to target.
- The goal is lipidome-level structural similarity, PCA, dendrogram clustering, or
  quality control rather than identifying lipids that distinguish study variables.
- A complete exhaustive feature search is required — this skill is a fast heuristic
  SFS terminated after √n rounds, not an exhaustive subset search.
tools:
- name: LipidSpace
  role: Standalone workstation application whose built-in feature analysis module
    runs the heuristic sequential forward selection of lipids per study variable and
    writes feature_analysis_results.csv.
  repo: https://github.com/lifs-tools/lipidspace
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:502683148f1033f54140da3c2786a8f538cc2dd1ce290f6fd2a14d30f4797957
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# lipid-feature-sequential-forward-selection

## Summary

LipidSpace's feature analysis and selection module applies a heuristic sequential forward selection (SFS) over lipid abundances to identify, per study variable, the lipid subset that best describes that variable, scored by AIC (and ROC accuracy for binary nominal variables).

## When to use

Use when a lipidomics abundance table is annotated with sample-associated study variables (e.g., BMI, tissue, condition) and you need the lipid subset that best distinguishes each variable.

## When NOT to use

- The dataset carries no sample-associated study variables (lipid abundances only) — there is nothing for the regression/classification models to target.
- The goal is lipidome-level structural similarity, PCA, dendrogram clustering, or quality control rather than identifying lipids that distinguish study variables.
- A complete exhaustive feature search is required — this skill is a fast heuristic SFS terminated after √n rounds, not an exhaustive subset search.

## Inputs

- Lipidome abundance table (flat or pivot csv/xlsx, or mzTab-M) with lipid species as rows/columns and samples annotated with categorical or numerical study variables
- Example dataset bundled with LipidSpace: examples/Example-Dataset.xlsx

## Outputs

- feature_analysis_results.csv containing, per study variable, the selected lipid subset, per-round AIC values, and (for binary nominal variables) ROC accuracies

## How to apply

Load a lipidome table (flat or pivot csv/xlsx, or mzTab-M) with lipid abundances plus attached study variables, treating lipids as features and study variables as targets. For each study variable, run sequential forward selection: start with zero features, fit a multiple linear regression for every single lipid, and score each model by its AIC; in each round, extend the current subset by testing all supersets containing the previously chosen features and keep the lowest-AIC model. Terminate selection after only the first √n steps (n = number of lipids/features) to limit computation and avoid overfitting. Per study variable, keep the model with the overall best (lowest) AIC; for binary nominal variables additionally evaluate separation quality via ROC accuracy. Write the selected lipid subsets, per-round AIC values, and (where applicable) ROC accuracies to feature_analysis_results.csv.

## Related tools

- **LipidSpace** (Standalone workstation application whose built-in feature analysis module runs the heuristic sequential forward selection of lipids per study variable and writes feature_analysis_results.csv.) — https://github.com/lifs-tools/lipidspace

## Evaluation signals

- feature_analysis_results.csv exists and contains, for every study variable, a selected lipid subset together with per-round AIC values.
- The number of SFS rounds per study variable does not exceed √n (n = total number of lipids/features), per the article's termination rule.
- For binary nominal study variables, ROC accuracy values are additionally present and the retained model is the one with the lowest AIC across rounds.
- Each round's retained superset contains the previously chosen feature(s) of the prior round (nested forward-selection invariant).
- Re-running on the bundled example dataset (examples/Example-Dataset.xlsx) reproduces the selected subsets for its attached study variables.

## Limitations

- SFS is a heuristic: it tests supersets of the current subset rather than all feature combinations, so the lowest-AIC subset found is not guaranteed to be globally optimal.
- Selection is truncated after the first √n steps to limit computational time and avoid overfitting, capping subset size relative to the number of lipids.
- ROC accuracy is only computed for binary nominal study variables; other variable types rely on AIC alone.
- Per the README, 'Complete feature analysis' additionally requires at least two study variables (it applies each variable's model to the others to compute inter-variable accuracies).

## Evidence

- [methods] SFS heuristic in methods: "We, therefore, applied an efficient, fast heuristic sequential forward selection (SFS) of features.20"
- [intro] Feature analysis purpose in intro: "The built-in feature analysis module assists in determining which lipids exert the highest effect in distinguishing between individual"
- [abstract] ML-based lipid subsets in abstract: "LipidSpace can determine lipid subsets across all lipidomes that describe these study variables well by utilizing machine-learning approaches."
- [readme] Study-variable requirement from README: "When your imported data contains at least two study variables, LipidSpace creates for each study variable a regression/classification model"
