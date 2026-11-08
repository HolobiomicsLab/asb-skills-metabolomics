---
name: random-forest-metabolomics-classification
description: Use when the analysis goal is supervised classification of metabolomics
  samples into class labels (e.g., treatment groups from a design file) and identification
  of which features differentiate those classes.
when_to_use_negative:
- The task is unsupervised or exploratory visualization (e.g., PCA, hierarchical cluster
  heatmap, modularity clustering) rather than classification against known class labels
- Only basic statistical testing is needed — SECIMTools' basic methods (PLS-DA, ANOVA,
  t-test, Kruskal-Wallis) are a better fit than random forest
- The input is raw spectra requiring peak picking / feature identification rather
  than a processed feature table
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SECIMTools
  role: Provides the random forest classification tool (standalone Python application
    and Galaxy wrapper) that trains the ensemble, generates per-sample predictions
    with vote shares, and emits per-feature importance scores
  repo: https://github.com/SECIMTools
- name: Python
  role: Runtime for the standalone SECIMTools RF tool; SECIMTools is packaged as a
    Python application and distributed via PyPI
  repo: https://pypi.python.org/pypi/secimtools
- name: Galaxy
  role: Web platform through which the SECIMTools RF tool can be run as a wrapped
    tool within integrated workflows
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:d3ff6cacbbbd59eb1517a68dbb6d2b66638bbcb0b85eaaeabd3019ff6dfa93d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# random-forest-metabolomics-classification

## Summary

Trains a random forest ensemble to classify metabolomics samples (e.g., treatment groups) from a wide-format feature table using SECIMTools' advanced classification tool, and ranks features by variable importance to identify the most differentiating metabolite features.

## When to use

Use when the analysis goal is supervised classification of metabolomics samples into class labels (e.g., treatment groups from a design file) and identification of which features differentiate those classes.

## When NOT to use

- The task is unsupervised or exploratory visualization (e.g., PCA, hierarchical cluster heatmap, modularity clustering) rather than classification against known class labels
- Only basic statistical testing is needed — SECIMTools' basic methods (PLS-DA, ANOVA, t-test, Kruskal-Wallis) are a better fit than random forest
- The input is raw spectra requiring peak picking / feature identification rather than a processed feature table

## Inputs

- Wide-format metabolomics feature table (samples as columns, features as rows, unique feature identifiers)
- Design file mapping each sample to a class label (e.g., treatment group)

## Outputs

- Trained random forest model file
- Per-sample predictions CSV containing predicted class and vote-share fields
- Per-feature importance scores CSV, rank-ordered by usefulness in the ensemble (VIF)

## How to apply

Load a wide-format feature table in which samples are columns and metabolomics features are rows with unique feature identifiers, plus a design file relating each sample to its class label (e.g., treatment group). Train the random forest ensemble as implemented by the SECIMTools RF tool, where each decision tree is built by randomly choosing both the samples and the features used to train it, with the design-file grouping variable serving as the class labels. Use the trained ensemble to assign a predicted class label to every sample and record the vote shares across trees. Assign an importance score to every feature, measuring how differentiating each feature is in the classification task, and rank-order the features by usefulness in the ensemble as summarized by the variable importance plot (VIF). Write the trained model file, the per-sample predictions CSV (predicted class and vote-share fields), and the per-feature importance scores CSV; SECIMTools runs both standalone as Python applications and wrapped for use in Galaxy.

## Related tools

- **SECIMTools** (Provides the random forest classification tool (standalone Python application and Galaxy wrapper) that trains the ensemble, generates per-sample predictions with vote shares, and emits per-feature importance scores) — https://github.com/SECIMTools
- **Python** (Runtime for the standalone SECIMTools RF tool; SECIMTools is packaged as a Python application and distributed via PyPI) — https://pypi.python.org/pypi/secimtools
- **Galaxy** (Web platform through which the SECIMTools RF tool can be run as a wrapped tool within integrated workflows)

## Evaluation signals

- The script runs end-to-end without errors
- All three output files exist: the model file, the per-sample predictions CSV, and the per-feature importance scores CSV
- The predictions CSV contains the predicted class field and vote-share fields for every sample
- The importance CSV contains per-feature importance score fields, with features rank-ordered by usefulness in the ensemble (as summarized by the variable importance plot, VIF)
- Predicted class labels are drawn from the design-file grouping variable (e.g., treatment groups) used as classification labels

## Limitations

- Random forest requires labeled samples — a design file with a grouping variable serving as class labels must be available; it is not applicable to unlabeled data
- The suite's random forest is one of several advanced classification options; support vector machines are the alternative advanced classification method in SECIMTools
- No version number for SECIMTools is given in the discussion context, and the GitHub reference is not resolved to a repository URL in the article text

## Evidence

- [abstract] advanced classification methods (random forest, support vector machines): "advanced classification methods (random forest, support vector machines)"
- [readme] SECIMTools python tools standalone and Galaxy: "The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
- [methods] SECIMTools packaged on PyPI: "we packaged all tools as a python package available from https://pypi.python.org/pypi/secimtools"
- [discussion] SECIMTools MIT license on GitHub: "SECIMTools suite takes advantage of the Galaxy interface and its code is available to the community under the terms of MIT license on GitHub"
