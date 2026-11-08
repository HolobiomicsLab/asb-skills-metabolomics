---
name: aic-based-regression-model-selection
description: Use when a lipidomics abundance table carries attached sample study variables
  (e.g., body mass index, tissue, condition) and you must identify which lipids exert
  the highest effect in distinguishing those variables.
when_to_use_negative:
- The imported dataset has fewer than two study variables — LipidSpace's regression/classification
  model feature analysis requires at least two study variables
- The goal is structural similarity comparison of lipidomes (MCS/Jaccard, PCA lipid
  space) rather than explaining study variables from lipid abundances
- The task is quality control or lipid name standardization rather than feature selection
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: LipidSpace
  role: Standalone workstation application whose feature analysis module runs the
    AIC-scored sequential forward selection and ROC evaluation over study variables
  repo: https://github.com/lifs-tools/lipidspace
- name: Goslin
  role: C++ library used by LipidSpace to parse and standardize lipid names so lipid
    features are consistently defined before selection
  repo: https://github.com/lifs-tools/cppgoslin
- name: openBLAS
  role: Library providing fast matrix multiplication underlying LipidSpace's linear-algebra
    computations
  repo: https://www.openblas.net/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:93d6400e97cbe25e48175e5b398777f0722ca32e4431968d227a11a39330d62b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - LipidSpace
  - openBLAS
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# aic-based-regression-model-selection

## Summary

Selects the lipid subset that best describes each study variable by heuristic sequential forward selection (SFS) over multiple linear regression models scored with the Akaike Information Criterion (AIC), with ROC accuracy added for binary nominal variables. Implemented as LipidSpace's built-in feature analysis module.

## When to use

Use when a lipidomics abundance table carries attached sample study variables (e.g., body mass index, tissue, condition) and you must identify which lipids exert the highest effect in distinguishing those variables.

## When NOT to use

- The imported dataset has fewer than two study variables — LipidSpace's regression/classification model feature analysis requires at least two study variables
- The goal is structural similarity comparison of lipidomes (MCS/Jaccard, PCA lipid space) rather than explaining study variables from lipid abundances
- The task is quality control or lipid name standardization rather than feature selection

## Inputs

- Lipidome abundance table (csv/xlsx, flat or pivot layout; lipids as features, samples as observations)
- Sample-associated study variables (nominal or numerical, e.g. body mass index, tissue, condition)
- Optional mzTab-M lipidomics data file (lipid names standardized via Goslin on import)

## Outputs

- Per-study-variable selected lipid subsets (best lowest-AIC model per variable)
- Per-round AIC values for each sequential forward selection step
- ROC accuracy values for binary nominal study variables
- feature_analysis_results.csv aggregating the above

## How to apply

Load a lipidome table (flat or pivot csv/xlsx, or mzTab-M) where lipids are features and categorical/numerical study variables are targets, e.g. the bundled 'examples/Example-Dataset.xlsx'. For each study variable, run sequential forward selection: start with zero features, fit a multiple linear regression for every single lipid, and score each model by AIC; in each round extend the current subset by testing all supersets containing the previously chosen features and keep the lowest-AIC model. Truncate the search after only the first √n rounds (n = number of lipids) to bound compute time and reduce overfitting. Per study variable, report the overall best (lowest) AIC model; for binary nominal variables additionally compute ROC accuracy of class separation, and write selected lipid subsets, per-round AIC values, and ROC accuracies to feature_analysis_results.csv. Rationale: exhaustive subset search is infeasible for hundreds of lipids, so the fast heuristic SFS trades optimality for tractability while still capturing the strongest predictors.

## Related tools

- **LipidSpace** (Standalone workstation application whose feature analysis module runs the AIC-scored sequential forward selection and ROC evaluation over study variables) — https://github.com/lifs-tools/lipidspace
- **Goslin** (C++ library used by LipidSpace to parse and standardize lipid names so lipid features are consistently defined before selection) — https://github.com/lifs-tools/cppgoslin
- **openBLAS** (Library providing fast matrix multiplication underlying LipidSpace's linear-algebra computations) — https://www.openblas.net/

## Examples

```
LipidSpace GUI: File → Import example dataset (examples/Example-Dataset.xlsx) → add study variables → Analysis → Complete feature analysis
```

## Evaluation signals

- AIC monotonically non-increasing (or consistently minimized) across forward-selection rounds for each study variable
- Number of selection rounds does not exceed √n, where n is the total number of lipid features
- Each study variable yields exactly one selected lipid subset plus its per-round AIC values in feature_analysis_results.csv
- For binary nominal variables, ROC accuracy is reported alongside the selected subset and is consistent with class separation quality
- Selected subsets remain valid across reanalysis runs of the same input table (deterministic heuristic behavior)

## Limitations

- SFS is a heuristic, not exhaustive: the greedy lowest-AIC subset is not guaranteed to be the global optimum
- Selection is truncated after only the first √n steps, so deeply interacting feature combinations may be missed
- ROC accuracy is only computed for binary nominal study variables; other variable types rely solely on AIC
- LipidSpace analyses cannot currently be stored or reloaded ('At the moment it is not possible to store or load an analysis')

## Evidence

- [abstract] LipidSpace can determine lipid subsets across all lipidomes that describe these study variables well by utilizing machine-learning approaches.: "LipidSpace can determine lipid subsets across all lipidomes that describe these study variables well by utilizing machine-learning approaches."
- [intro] The built-in feature analysis module assists in determining which lipids exert the highest effect in distinguishing between individual study variables or combinations thereof: "The built-in feature analysis module assists in determining which lipids exert the highest effect in distinguishing between individual study variables or combinations thereof"
- [methods] We, therefore, applied an efficient, fast heuristic sequential forward selection (SFS) of features.20: "We, therefore, applied an efficient, fast heuristic sequential forward selection (SFS) of features.20"
- [readme] When your imported data contains at least two study variables, LipidSpace creates for each study variable a regression/classification model and applies this model on all remaining study variables to compute the accuracy.: "When your imported data contains at least two study variables, LipidSpace creates for each study variable a regression/classification model and applies this model on all remaining study variables to"
