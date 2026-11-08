---
name: ensemble-model-training-and-prediction
description: Use when a wide-format metabolomics feature table (samples as columns,
  features as rows with unique identifiers) is paired with a design file assigning
  samples to class labels (e.g. treatment group) and the goal is supervised classification
  plus feature ranking.
when_to_use_negative:
- The task is unsupervised or exploratory (e.g. visualization via PCA or hierarchical
  cluster heatmap) rather than supervised classification against design-file class
  labels
- Only basic statistical testing is needed, for which SECIMTools provides PLS-DA,
  ANOVA, t-test, or Kruskal-Wallis instead
- The input is raw instrument data requiring preprocessing (peak picking, imputation,
  blank feature filtering) rather than a feature-level table
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SECIMTools
  role: Provides the random forest tool used to train the ensemble, predict sample
    classes with vote shares, and produce per-feature importance scores; available
    standalone and wrapped for Galaxy
  repo: https://github.com/SECIMTools
- name: Galaxy
  role: Web platform through which the SECIMTools random forest tool can be wrapped
    and run as part of integrated workflows
- name: Python
  role: Language in which SECIMTools' random forest classification tool is implemented
    and run standalone
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:08d6876838e8603a45bf4288dd592a19a455aa1fe15f41f97e0dd928c4446d54
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ensemble-model-training-and-prediction

## Summary

Train a random forest ensemble to classify metabolomics samples by a design-file class label, then use the forest to predict class membership (with per-tree vote shares) and rank features by variable importance, as implemented in SECIMTools' random forest tool.

## When to use

Use when a wide-format metabolomics feature table (samples as columns, features as rows with unique identifiers) is paired with a design file assigning samples to class labels (e.g. treatment group) and the goal is supervised classification plus feature ranking.

## When NOT to use

- The task is unsupervised or exploratory (e.g. visualization via PCA or hierarchical cluster heatmap) rather than supervised classification against design-file class labels
- Only basic statistical testing is needed, for which SECIMTools provides PLS-DA, ANOVA, t-test, or Kruskal-Wallis instead
- The input is raw instrument data requiring preprocessing (peak picking, imputation, blank feature filtering) rather than a feature-level table

## Inputs

- Wide-format metabolomics feature table (samples as columns, metabolomics features as rows with unique feature identifiers)
- Design file mapping each sample to its class label (e.g. treatment group), used as the grouping variable

## Outputs

- Trained model file
- Per-sample predictions CSV containing predicted class and vote-share fields
- Per-feature importance scores CSV (rank-orderable, summarized in the variable importance plot, VIF)

## How to apply

Load the wide-format dataset in which samples are columns and metabolomics features are rows with unique feature identifiers, together with the design file relating each sample to its class label. Train the random forest ensemble as implemented by the SECIMTools RF tool (standalone Python or Galaxy-wrapped), creating each tree by randomly choosing both the samples and the features used to train it, with the design-file grouping variable as the class labels. Use the trained ensemble to assign a predicted class to every sample and record the vote shares across trees. Compute a per-feature importance score measuring how differentiating each feature is, and rank-order features by usefulness, summarized in the variable importance plot (VIF). Rationale: SECIMTools lists random forest among its advanced classification methods, alongside support vector machines, as a complement to its basic statistical methods (PLS-DA, ANOVA, t-test, Kruskal-Wallis).

## Related tools

- **SECIMTools** (Provides the random forest tool used to train the ensemble, predict sample classes with vote shares, and produce per-feature importance scores; available standalone and wrapped for Galaxy) — https://github.com/SECIMTools
- **Galaxy** (Web platform through which the SECIMTools random forest tool can be wrapped and run as part of integrated workflows)
- **Python** (Language in which SECIMTools' random forest classification tool is implemented and run standalone)

## Evaluation signals

- The script runs end-to-end and all three output files exist: the trained model file, the predictions CSV, and the importance scores CSV
- The predictions CSV contains both the predicted class field and the vote-share fields for every sample
- The importance CSV contains importance score fields for every feature, and features can be rank-ordered by score
- Predicted class labels come from the design-file grouping variable (e.g. treatment group), so every sample receives a label from that set
- Feature importance ranks can be summarized in the variable importance plot (VIF)

## Limitations

- No version number for SECIMTools or Galaxy is given in the discussion; the GitHub reference [27] is not resolved to a repository URL in the text
- The article/README provide no concrete command-line or parameter invocation for the RF tool, only that it exists as a standalone and Galaxy-wrapped tool
- Random forest is one of several advanced classification/variable selection options; if a sparse linear model is preferred, SECIMTools also offers LASSO and Elastic Net

## Evidence

- [abstract] advanced classification methods (random forest, support vector machines): "advanced classification methods (random forest, support vector machines)"
- [readme] SECIMTools RF tool availability: "The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
- [methods] Packaging and license: "we packaged all tools as a python package available from https://pypi.python.org/pypi/secimtools"
- [intro] Standalone or Galaxy operation: "SECIMTools project aims to develop a suite of tools for processing of metabolomics data, which can be run in a standalone mode or via Galaxy"
