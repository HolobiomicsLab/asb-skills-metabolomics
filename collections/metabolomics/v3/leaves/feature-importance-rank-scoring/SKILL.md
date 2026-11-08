---
name: feature-importance-rank-scoring
description: Use when a wide-format metabolomics feature table plus a design file
  with class labels are available and the goal is to identify which features differentiate
  sample groups via random-forest variable importance.
when_to_use_negative:
- Input is a peak list or raw instrument data rather than a preprocessed feature table
  — SECIMTools expects features as the starting point; peak picking/feature identification
  is done upstream (e.g. by Galaxy-M or Workflow4Metabolomics).
- The task is regression or unsupervised ordination (PCA, hierarchical cluster heatmap,
  modularity clustering) rather than class-label classification with a design-file
  grouping variable.
- The goal is hypothesis testing on group differences (ANOVA, t-test, Kruskal-Wallis)
  rather than ensemble-based feature differentiation.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: SECIMTools
  role: Provides the random-forest classification tool (standalone or Galaxy-wrapped)
    that trains the tree ensemble on wide-format feature data and emits per-sample
    predictions and per-feature importance scores
  repo: https://github.com/SECIMTools
- name: Python
  role: Implementation language of SECIMTools; the RF tool is available as a standalone
    Python application
  repo: https://pypi.python.org/pypi/secimtools
- name: Galaxy
  role: Web platform wrapping the SECIMTools RF tool so the importance-ranking workflow
    can be run and chained via Galaxy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:6cdc39be99031512c3e77982ace1c14c022f4d1fda87e1e52f07d5eed4390e19
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# feature-importance-rank-scoring

## Summary

Assigns an importance score to every metabolomics feature in a random-forest classifier, measuring how differentiating each feature is, and rank-orders features by usefulness in the ensemble (summarized by the variable importance plot, VIF).

## When to use

Use when a wide-format metabolomics feature table plus a design file with class labels are available and the goal is to identify which features differentiate sample groups via random-forest variable importance.

## When NOT to use

- Input is a peak list or raw instrument data rather than a preprocessed feature table — SECIMTools expects features as the starting point; peak picking/feature identification is done upstream (e.g. by Galaxy-M or Workflow4Metabolomics).
- The task is regression or unsupervised ordination (PCA, hierarchical cluster heatmap, modularity clustering) rather than class-label classification with a design-file grouping variable.
- The goal is hypothesis testing on group differences (ANOVA, t-test, Kruskal-Wallis) rather than ensemble-based feature differentiation.

## Inputs

- wide-format metabolomics feature table (samples as columns, features as rows with unique feature identifiers)
- design file relating each sample to its class label (grouping variable, e.g. treatment group)

## Outputs

- trained random-forest model file
- per-sample predictions CSV with predicted class and vote-share fields
- per-feature importance scores CSV (rank-ordered by usefulness in the ensemble)

## How to apply

Load the SECIMTools example wide-format dataset where samples are columns and features are rows with unique feature identifiers, together with the design file mapping each sample to its class label (e.g. treatment group). Train the random forest as implemented by the SECIMTools RF tool, where each decision tree is built from randomly chosen samples and features, with the design-file grouping variable serving as class labels. Use the trained ensemble to assign predicted class labels and record vote shares across trees. Then compute a per-feature importance score measuring how differentiating each feature is in the classification task and rank-order features by usefulness in the ensemble, as summarized by the variable importance plot (VIF). Rationale: SECIMTools provides random forest as an advanced classification method alongside variable-selection tools (LASSO, Elastic Net), so importance ranking can be cross-checked against penalized-regression feature selection.

## Related tools

- **SECIMTools** (Provides the random-forest classification tool (standalone or Galaxy-wrapped) that trains the tree ensemble on wide-format feature data and emits per-sample predictions and per-feature importance scores) — https://github.com/SECIMTools
- **Python** (Implementation language of SECIMTools; the RF tool is available as a standalone Python application) — https://pypi.python.org/pypi/secimtools
- **Galaxy** (Web platform wrapping the SECIMTools RF tool so the importance-ranking workflow can be run and chained via Galaxy)

## Evaluation signals

- Script runs end-to-end and all three output files exist (model file, predictions CSV, importance CSV).
- Predictions CSV contains prediction and vote-share fields for every sample.
- Importance CSV contains importance score fields for every feature, with a complete rank ordering.
- Every sample in the design file receives exactly one predicted class label consistent with the grouping-variable levels.
- Feature importance ranking is reproducible when re-run with the same random-forest seed/sample-feature resampling configuration.

## Limitations

- Tree ensembles are built by randomly choosing both samples and features, so importance scores are stochastic and depend on the ensemble draw; no seed or tree-count defaults are stated in the article or README.
- Importance ranking depends on the quality of the design-file class labels; misassigned groups will propagate to the ranking.
- The article notes that untargeted metabolomics is a relatively new field with few analysis tools, so best-practice thresholds for feature-importance cutoffs are not established in this context.

## Evidence

- [abstract] advanced classification methods (random forest, support vector machines): "advanced classification methods (random forest, support vector machines)"
- [readme] a set of python tools that are available both as standalone and wrapped for use in Galaxy: "The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
- [readme] advanced variable selection tools (least absolute shrinkage and selection operator LASSO and Elastic Net): "and advanced variable selection tools (least absolute shrinkage and selection operator LASSO and Elastic Net)."
- [methods] SECIMTools is open source, the code, is available on GitHub using the MIT license: "SECIMTools is open source, the code, is available on GitHub using the MIT license"
