---
name: metabolomics-data-scaling
description: Use when a normalized, transformed metabolomics table (e.g., post Log10/Log2/square-root/cube-root
  transformation) still spans a large dynamic range across metabolites and needs variance-scaled
  columns before statistical analysis or PCA-based data treatment evaluation.
when_to_use_negative:
- The table has not yet been normalized and transformed — Viime's scaling is the final
  data treatment step and assumes prior normalization and Log/square-root/cube-root
  transformation
- The goal is upstream error correction, imputation, or normalization (e.g., MCAR/MNAR
  imputation or reference-sample normalization) — those are earlier Viime workflow
  steps
- The input is raw vendor output or non-concentration data (e.g., non-numeric concentration
  columns), which Viime flags as an error before data treatment proceeds
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: Viime
  role: Provides the scaling step of the data treatment workflow with its five scaling
    options; scaling is applied within Viime's backend
  repo: https://github.com/girder/viime
- name: Pandas
  role: Used by the Viime backend for parsing the metabolomics table into a DataFrame
    before and after scaling
- name: Scikit-learn
  role: Supplies the scaling utilities for the variance-scaling options (autoscaling,
    Pareto, range, vast, level) in the Viime backend
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:cdaeea5fc490c9ae170ce4fa0b9583c7caefdd82f2413c8e94187c27022dc47a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Pandas
  - Scikit-learn
  - Viime
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metabolomics-data-scaling

## Summary

Final data treatment step in the Viime metabolomics workflow that addresses the large dynamic range of metabolomics data by scaling the variance of each metabolite column. Offers five scaling options: autoscaling, Pareto scaling, range scaling, vast scaling, and level scaling, implemented via scikit-learn on previously normalized and transformed data.

## When to use

Use when a normalized, transformed metabolomics table (e.g., post Log10/Log2/square-root/cube-root transformation) still spans a large dynamic range across metabolites and needs variance-scaled columns before statistical analysis or PCA-based data treatment evaluation.

## When NOT to use

- The table has not yet been normalized and transformed — Viime's scaling is the final data treatment step and assumes prior normalization and Log/square-root/cube-root transformation
- The goal is upstream error correction, imputation, or normalization (e.g., MCAR/MNAR imputation or reference-sample normalization) — those are earlier Viime workflow steps
- The input is raw vendor output or non-concentration data (e.g., non-numeric concentration columns), which Viime flags as an error before data treatment proceeds

## Inputs

- Normalized and transformed metabolomics data table (.csv or .xlsx), e.g., one previously processed with Log10/Log2/square root/cube root transformation

## Outputs

- Variance-scaled metabolomics data table (CSV), with each metabolite column scaled per the chosen option (autoscale, Pareto, range, vast, or level)

## How to apply

Load the metabolomics data table — already normalized (reference sample, sum scaling, or weight/volume normalization) and transformed (Log10, Log2, square root, or cube root) — as a DataFrame with pandas. Apply the selected scaling option by scaling the variance of each metabolite column: autoscaling (unit variance), Pareto scaling, range scaling, vast scaling, or level scaling, using scikit-learn's scaling utilities as the Viime backend does. Choose the option interactively: Viime pairs the data treatment pipeline with interactive PCA score and loadings plots to examine similarity and dissimilarity of individual groups, so assess grouping structure after each candidate scaling. Write the scaled data table back out as a CSV file, then proceed to downstream analyses such as Wilcoxon rank sum tests, multivariate ANOVA, heatmaps, or volcano plots.

## Related tools

- **Viime** (Provides the scaling step of the data treatment workflow with its five scaling options; scaling is applied within Viime's backend) — https://github.com/girder/viime
- **Pandas** (Used by the Viime backend for parsing the metabolomics table into a DataFrame before and after scaling)
- **Scikit-learn** (Supplies the scaling utilities for the variance-scaling options (autoscaling, Pareto, range, vast, level) in the Viime backend)

## Examples

```
import pandas as pd; from sklearn.preprocessing import scale; df = pd.read_csv('metabolites_transformed.csv'); scaled = pd.DataFrame(scale(df), columns=df.columns); scaled.to_csv('metabolites_scaled.csv', index=False)
```

## Evaluation signals

- Each metabolite column exhibits the variance behavior expected of the chosen scaling method (e.g., autoscaling yields unit variance per column)
- The scaled table retains the same row/column structure (samples and metabolite columns) as the input table
- Interactive PCA score and loadings plots show sensible group similarity/dissimilarity after scaling, consistent with Viime's data-treatment evaluation practice
- Output is a valid CSV that downstream Viime steps (univariate Wilcoxon rank sum, multivariate ANOVA, heatmaps) can consume without schema errors
- Scaling was applied only to metabolite concentration columns, not to the primary ID, factor, group, or masked/hidden designation columns

## Limitations

- Scaling addresses large dynamic range but is one of several sequential data treatment steps; it does not correct errors or handle missing data (imputation is automatic and separate)
- Choice among the five scaling options is analyst-driven; Viime surfaces it via interactive PCA inspection rather than prescribing a default
- The workflow expects data already after preprocessing and annotation; annotation is not required but upstream preprocessing is assumed

## Evidence

- [abstract] Scaling options: "The options here include autoscaling, Pareto scaling, range scaling, vast scaling and level scaling."
- [abstract] Scaling rationale: [evidence span withheld — anchor status: not_found]
- [abstract] Transformation prerequisites: "The options in this step are Log10, Log2, square root and cube root."
- [abstract] PCA-guided evaluation: "A very important feature of the whole data treatment process is the interactive use of principal component analysis (PCA) to examine the similarity and dissimilarity of individual groups"
- [abstract] Backend tooling: "Data processing is done by a combination of Scikit-learn for common statistical algorithms and R packages for specialized algorithms."
- [abstract] Workflow scope: "The intent of Viime package is to pick up the workflow after preprocessing and annotation and go all the way through statistical analysis and visualization."
