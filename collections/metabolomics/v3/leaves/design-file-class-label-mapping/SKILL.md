---
name: design-file-class-label-mapping
description: Use when preparing a wide-format feature table for SECIMTools supervised
  tools — random forest, SVM, LASSO/Elastic Net — and a design file relating each
  sample to its group must be joined to the sample columns.
when_to_use_negative:
- The analysis is a class-label-free step such as blank feature filtering, retention
  time diagnostics, or run order regression, where the grouping variable is not used
  to supervise the tool
- Sample identifiers in the feature table do not match design-file sample IDs, so
  the join would silently mislabel samples rather than map them
- The labels to attach are continuous values rather than a discrete grouping variable,
  since the RF classifier and the binary-label Elastic Net fit expect categorical
  classes
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: SECIMTools
  role: Supplies the wide-format example dataset, the design file, and the classification/variable-selection
    tools (RF, LASSO/Elastic Net) that consume the mapped class labels
  repo: https://github.com/SECIMTools
- name: Python
  role: Runtime in which the standalone SECIMTools applications perform the design-file-to-sample
    label mapping
- name: Galaxy
  role: Platform through which SECIMTools tools, including design-file inputs, can
    be run wrapped for use in Galaxy
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:75f6393d43ec792a7f5ff8d1bba94e6891f62638e1f68fc9b8e8a197d6c65a0e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# design-file-class-label-mapping

## Summary

Maps each sample in a SECIMTools wide-format metabolomics feature table (samples in columns, features in rows with unique feature identifiers) to its class label via the design file, so the user-specified grouping variable (e.g. treatment group) can serve as the class labels for downstream classification and variable-selection tools.

## When to use

Use when preparing a wide-format feature table for SECIMTools supervised tools — random forest, SVM, LASSO/Elastic Net — and a design file relating each sample to its group must be joined to the sample columns.

## When NOT to use

- The analysis is a class-label-free step such as blank feature filtering, retention time diagnostics, or run order regression, where the grouping variable is not used to supervise the tool
- Sample identifiers in the feature table do not match design-file sample IDs, so the join would silently mislabel samples rather than map them
- The labels to attach are continuous values rather than a discrete grouping variable, since the RF classifier and the binary-label Elastic Net fit expect categorical classes

## Inputs

- SECIMTools bundled example wide-format dataset (samples in columns, metabolomics features in rows with unique feature identifiers)
- design file relating each sample to its class label (e.g. treatment group)
- user-specified grouping variable name selecting which design-file column supplies the class labels

## Outputs

- sample-to-class-label mapping attached to the sample columns of the wide-format table, ready for the random forest classifier or the LASSO/Elastic Net variable-selection tool
- class labels consumed downstream to produce the per-sample predictions CSV (predicted class and vote-share fields) and the per-feature importance scores CSV

## How to apply

Load the SECIMTools bundled example wide-format dataset, in which samples are columns and metabolomics features are rows with unique feature identifiers, together with the design file that relates each sample to its class label. Choose the user-specified grouping variable (e.g. treatment group) and attach its labels to the sample columns so the design-file grouping variable serves as the class labels for the classification task. For Elastic Net, the attached labels must be binary before fitting logistic regression with shrinkage parameter alpha = 0.5 (the tool default within the [0;1] range, where alpha = 1 gives LASSO and alpha = 0 gives ridge). Verify that every sample column resolves to exactly one design-file row before handing the mapped table to the RF ensemble or the LASSO/Elastic Net wrapper, since vote-share and per-feature importance outputs depend on correctly labeled samples.

## Related tools

- **SECIMTools** (Supplies the wide-format example dataset, the design file, and the classification/variable-selection tools (RF, LASSO/Elastic Net) that consume the mapped class labels) — https://github.com/SECIMTools
- **Python** (Runtime in which the standalone SECIMTools applications perform the design-file-to-sample label mapping)
- **Galaxy** (Platform through which SECIMTools tools, including design-file inputs, can be run wrapped for use in Galaxy)

## Evaluation signals

- Every sample column in the wide-format table resolves to exactly one design-file row, with no missing or unmatched sample IDs
- Each sample carries exactly one class label drawn from the user-specified grouping variable (e.g. treatment group)
- The downstream per-sample predictions CSV contains predicted class and vote-share fields for every labeled sample
- The per-feature importance CSV contains importance score fields for every feature row in the input table
- For Elastic Net, the labels are binary and elastic_net_feature_selection.csv contains a coefficient and a selected/not-selected flag for every input feature

## Limitations

- The grouping variable must be present in the design file and align with sample column names; the article does not document an error mode for mismatched sample identifiers
- Elastic Net class-label mapping assumes a binary outcome, consistent with the logistic-regression formulation of the SECIMTools LASSO/Elastic Net tool
- No version number for SECIMTools or Galaxy is given, so the exact design-file schema expected by the tools is not fully pinned down by the article

## Evidence

- [other] design-file grouping variable as class labels: [evidence span withheld — anchor status: not_found]
- [other] attach binary labels from grouping variable: [evidence span withheld — anchor status: not_found]
- [other] wide-format dataset and design file: [evidence span withheld — anchor status: not_found]
- [abstract] classification methods supported: "advanced classification methods (random forest, support vector machines)"
- [abstract] variable selection tools supported: "advanced variable selection tools (least absolute shrinkage and selection operator LASSO and Elastic Net)"
- [readme] standalone and Galaxy availability: "SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
