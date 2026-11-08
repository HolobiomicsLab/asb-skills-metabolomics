---
name: elastic-net-penalized-logistic-regression
description: Use when a wide-format feature table (samples in columns, metabolomics
  features in rows with unique feature identifiers) plus a design file with a binary
  grouping variable is available, and the goal is penalized feature selection rather
  than plain significance testing.
when_to_use_negative:
- The input is raw instrument data (spectra/peaks) rather than a preprocessed wide-format
  feature table — SECIMTools' statistical tools operate downstream of feature identification.
- The outcome is not a binary class label from the design file's grouping variable
  (the tool is framed around binary classification labels).
- Pure L1 sparsity is required without ridge mixing — the article's LASSO tool (alpha
  = 1) is the intended choice instead.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: SECIMTools
  role: Provides the LASSO/Elastic Net variable-selection tool (Python applications,
    also Galaxy-wrapped) that this skill reproduces; tool default alpha = 0.5 and
    outputs match Additional file 2, Figures S24–S25
  repo: https://github.com/SECIMTools
- name: Galaxy
  role: Web platform through which the SECIMTools Elastic Net tool is deployed and
    from which the resulting feature-selection table is produced
- name: Python
  role: Implementation language for the standalone SECIMTools tools and for scripting
    the Elastic Net fit and per-feature coefficient extraction
  repo: https://pypi.python.org/pypi/secimtools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:7bdcaa81b47dc0819dd059fcf07acea1bc04669befcde05af4be4bf2af26a14a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# elastic-net-penalized-logistic-regression

## Summary

Fits an Elastic Net penalized logistic regression over a metabolomics feature table to select the best subset of discriminating features, following the SECIMTools LASSO/Elastic Net variable-selection tool (a Galaxy-wrapped implementation). Elastic Net blends L1 (LASSO) and L2 (ridge) penalties so correlated metabolomics features are selected in groups rather than arbitrarily.

## When to use

Use when a wide-format feature table (samples in columns, metabolomics features in rows with unique feature identifiers) plus a design file with a binary grouping variable is available, and the goal is penalized feature selection rather than plain significance testing.

## When NOT to use

- The input is raw instrument data (spectra/peaks) rather than a preprocessed wide-format feature table — SECIMTools' statistical tools operate downstream of feature identification.
- The outcome is not a binary class label from the design file's grouping variable (the tool is framed around binary classification labels).
- Pure L1 sparsity is required without ridge mixing — the article's LASSO tool (alpha = 1) is the intended choice instead.

## Inputs

- Wide-format metabolomics feature table (samples in columns, features in rows, unique feature identifiers)
- Design file with sample metadata including a user-specified binary grouping variable
- Shrinkage parameter alpha (default 0.5, valid range [0;1])

## Outputs

- elastic_net_feature_selection.csv — Galaxy-compatible table with one row per input feature: feature ID, fitted coefficient, and selected/not-selected flag

## How to apply

Load the SECIMTools-style wide-format dataset and its design file, and attach the binary class labels from the user-specified grouping variable to the sample columns. Fit a logistic regression with Elastic Net shrinkage of the features against the binary labels, mirroring the SECIMTools LASSO/Elastic Net tool, which is a wrapper for the inventors' GNU-licensed R code. Set the shrinkage parameter alpha = 0.5 (the tool default) within the [0;1] range, where alpha = 1 reduces to LASSO and alpha = 0 to ridge regression. Extract the fitted coefficient for every feature and mark membership in the best subset selected for the given alpha as a selected/not-selected flag. Write the per-feature results (feature ID, coefficient, selection flag) as a Galaxy-compatible table named elastic_net_feature_selection.csv, mirroring the tool's outputs illustrated in Additional file 2, Figures S24 and S25.

## Related tools

- **SECIMTools** (Provides the LASSO/Elastic Net variable-selection tool (Python applications, also Galaxy-wrapped) that this skill reproduces; tool default alpha = 0.5 and outputs match Additional file 2, Figures S24–S25) — https://github.com/SECIMTools
- **Galaxy** (Web platform through which the SECIMTools Elastic Net tool is deployed and from which the resulting feature-selection table is produced)
- **Python** (Implementation language for the standalone SECIMTools tools and for scripting the Elastic Net fit and per-feature coefficient extraction) — https://pypi.python.org/pypi/secimtools

## Evaluation signals

- The script runs without error and elastic_net_feature_selection.csv exists.
- The output table contains a coefficient value and a selected/not-selected flag for every input feature row.
- Output rows correspond one-to-one to the feature identifiers of the input wide-format dataset (no dropped or added features).
- The shrinkage parameter alpha used lies in [0;1]; alpha = 0.5 reproduces the tool default, alpha = 1 gives pure LASSO, alpha = 0 gives ridge regression.
- Selected features are those with non-zero/flagged membership in the best subset for the chosen alpha, consistent with the tool outputs shown in Additional file 2, Figures S24 and S25.

## Limitations

- The tool is a wrapper for the inventors' GNU-licensed R code, so exact behavior depends on the wrapped implementation rather than a native Python reimplementation.
- The article gives no version number for SECIMTools or Galaxy, so reproducing results requires pinning the tool version independently.
- Feature tables must first be preprocessed (e.g., imputation, normalization/re-scaling via SECIMTools tools) before variable selection; missing-value handling is not part of this skill.

## Evidence

- [abstract] SECIMTools includes variable selection tools (LASSO and Elastic Net): "advanced variable selection tools (least absolute shrinkage and selection operator LASSO and Elastic Net)"
- [readme] SECIMTools is a Python package installable from PyPI and usable standalone or via Galaxy: "The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
- [methods] SECIMTools is open source and available on GitHub under MIT license: "SECIMTools is open source, the code, is available on GitHub using the MIT license"
