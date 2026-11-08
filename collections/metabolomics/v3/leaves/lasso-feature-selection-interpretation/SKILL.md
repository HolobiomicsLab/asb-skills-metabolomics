---
name: lasso-feature-selection-interpretation
description: Decision support for selecting the subset of metabolomics features that
  discriminate between sample groups in a wide-format feature table.
when_to_use_negative:
- The goal is peak picking or preprocessing rather than statistical variable selection
  — use Galaxy-M or Workflow4Metabolomics for upstream feature identification instead.
- Only one sample group exists (no binary grouping variable), so penalized discriminative
  regression has no class to fit against.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SECIMTools
  role: Provides the LASSO variable selection tool (a Python/Galaxy-wrapped implementation
    of the inventors' GNU-licensed R code) operating on wide-format metabolomics feature
    tables
  repo: https://github.com/SECIMTools
- name: Galaxy
  role: Deployment platform that runs SECIMTools' LASSO tool and produces the per-feature
    coefficient/selection-flag table
  repo: https://github.com/SECIMTools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:7d3747df426a16d01b0ff73116d689e42d5aa786381ef86e728618eec514e9dd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# lasso-feature-selection-interpretation

## Summary

SECIMTools provides LASSO (and its companion Elastic Net) as advanced variable selection methods for metabolomics feature tables, using penalized regression to identify the best subset of discriminating features for a grouping variable.

## When to use

Decision support for selecting the subset of metabolomics features that discriminate between sample groups in a wide-format feature table.

## When NOT to use

- The goal is peak picking or preprocessing rather than statistical variable selection — use Galaxy-M or Workflow4Metabolomics for upstream feature identification instead.
- Only one sample group exists (no binary grouping variable), so penalized discriminative regression has no class to fit against.

## Inputs

- SECIMTools bundled example wide-format metabolomics dataset (samples in columns, features in rows with unique feature identifiers)
- SECIMTools design file carrying the sample-grouping variable

## Outputs

- Per-feature table (feature ID, fitted coefficient, selected/not-selected flag) e.g. lasso_feature_selection.csv, Galaxy-compatible

## How to apply

Load SECIMTools' bundled example wide-format dataset (samples in columns, metabolomics features in rows with unique feature identifiers) and its design file, attaching the binary class labels from the user-specified grouping variable to the sample columns. Fit logistic regression with LASSO (alpha = 1, the fully penalized end of the [0;1] shrinkage parameter range) against the binary labels — SECIMTools wraps the inventors' GNU-licensed R code for this. Extract the fitted coefficient for every feature and flag membership in the best subset of variables selected for the given alpha as a selected/not-selected flag. Write the per-feature results (feature ID, coefficient, selection flag) as a Galaxy-compatible table, as illustrated in Additional file 2, Figures S24 and S25.

## Related tools

- **SECIMTools** (Provides the LASSO variable selection tool (a Python/Galaxy-wrapped implementation of the inventors' GNU-licensed R code) operating on wide-format metabolomics feature tables) — https://github.com/SECIMTools
- **Galaxy** (Deployment platform that runs SECIMTools' LASSO tool and produces the per-feature coefficient/selection-flag table) — https://github.com/SECIMTools

## Evaluation signals

- The script runs without error and produces a per-feature table listing coefficient and selection-flag fields for every input feature row.
- Every feature ID in the input wide-format table appears exactly once in the output table.
- The selection flag is binary (selected / not-selected) for each feature, and selected features form a reduced subset relative to the full feature set.
- Coefficients are continuous values consistent with penalized logistic regression output (many unselected features carry exactly zero coefficient).

## Limitations

- LASSO's shrinkage parameter alpha is fixed at 1 for this tool; Elastic Net (alpha = 0.5 default) is a separate tool for mixing ridge and LASSO penalties.
- The tool expects binary class labels from a single grouping variable; multi-class grouping is not supported by this implementation.
- The article does not report version numbers for SECIMTools or Galaxy, so reproducibility requires pinning the commit or package version separately.

## Evidence

- [abstract] SECIMTools includes variable selection tools (LASSO and Elastic Net): "advanced variable selection tools (least absolute shrinkage and selection operator LASSO and Elastic Net)"
- [other] LASSO uses shrinkage of the features against binary labels, following SECIMTools LASSO/Elastic Net tool (a wrapper for the inventors' GNU-licensed R code): [evidence span withheld — anchor status: not_found]
- [abstract] SECIMTools is a suite of Python applications available standalone or in Galaxy: "SECIMTools (SouthEast Center for Integrated Metabolomics) is a set of Python applications that are available both as standalone tools and wrapped for use in Galaxy."
- [readme] SECIMTools code is available on GitHub under MIT license: "The project is licensed under MIT license"
