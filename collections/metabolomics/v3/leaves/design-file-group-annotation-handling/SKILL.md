---
name: design-file-group-annotation-handling
description: Use when a downstream per-feature analysis (ANOVA, t-test, Kruskal-Wallis)
  requires samples to be assigned to treatment groups via a design file paired with
  a wide-format dataset.
when_to_use_negative:
- Input data are raw instrument files at the peak-picking/feature-identification stage
  — group annotation applies after preprocessing produces a feature table
- The analysis is unsupervised (e.g., PCA, hierarchical cluster heatmap, modularity
  clustering) and does not require treatment group labels
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: SECIMTools
  role: Provides the wide-format dataset/design-file standard data formats consumed
    by the ANOVA tool and other group-aware tools (Threshold Based Flags, Imputation
    by group mean/median)
  repo: https://github.com/SECIMTools
- name: Galaxy
  role: Web platform in which SECIMTools design-file-based tools can be run and chained
    into integrated workflows
- name: Python
  role: Implementation language of the SECIMTools standalone applications that read
    the design file
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:8d168d746698442b43e42db80eafcb8b5ab6e91ed00d3f55638d622078c7c280
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# design-file-group-annotation-handling

## Summary

Load and align a metabolomics wide-format feature table (samples as columns, features as rows with unique identifiers) with a design file that maps each sample to its treatment group annotation, so that per-feature statistical analyses such as single-factor fixed ANOVA in SECIMTools can be grouped and computed correctly.

## When to use

Use when a downstream per-feature analysis (ANOVA, t-test, Kruskal-Wallis) requires samples to be assigned to treatment groups via a design file paired with a wide-format dataset.

## When NOT to use

- Input data are raw instrument files at the peak-picking/feature-identification stage — group annotation applies after preprocessing produces a feature table
- The analysis is unsupervised (e.g., PCA, hierarchical cluster heatmap, modularity clustering) and does not require treatment group labels

## Inputs

- Wide-format metabolomics feature table (samples as columns, features as rows with unique feature identifiers), e.g. the bundled SECIMTools example dataset
- Design file mapping each sample to its treatment group annotation

## Outputs

- Sample-to-group annotation aligned to feature table columns, ready for per-feature grouped analysis
- Per-feature results tables keyed by treatment group (group means, F-statistics, p-values, significance flags), e.g. anova_results.csv

## How to apply

Take the bundled SECIMTools example wide-format dataset, in which samples are columns and metabolomics features are rows with unique feature identifiers, together with its design file that maps each sample to its treatment group annotation. Before fitting any per-feature model, join each sample column to its group label from the design file so every feature row can be stratified by treatment group. This annotation step is what enables the fixed-effects ANOVA to compare group means, compute all pairwise contrasts, and derive F-statistics per feature. Keep the standard SECIMTools data formats so utilities allow arbitrary linkages between tools for novel workflow designs; the same design-file mapping feeds QC flag tools (e.g., Threshold Based Flags evaluates features within a given group) and imputation (group mean or group median).

## Related tools

- **SECIMTools** (Provides the wide-format dataset/design-file standard data formats consumed by the ANOVA tool and other group-aware tools (Threshold Based Flags, Imputation by group mean/median)) — https://github.com/SECIMTools
- **Galaxy** (Web platform in which SECIMTools design-file-based tools can be run and chained into integrated workflows)
- **Python** (Implementation language of the SECIMTools standalone applications that read the design file)

## Evaluation signals

- Every sample column in the wide-format table has exactly one matching treatment group entry in the design file (no unmatched or duplicated samples)
- Per-feature ANOVA output (anova_results.csv) contains group means, F-statistics, p-values for all pairwise contrasts, and binary significance flag columns
- Group-wise summaries (means/medians) computed after annotation are consistent with the design file's group assignments
- The files can be imported into Galaxy and used in SECIMTools without format conversion, per the standard data formats requirement

## Limitations

- The design file must map every sample to exactly one treatment group; mis-annotated samples propagate into group means, contrasts, and flags
- No version number for SECIMTools or Galaxy is given in the discussion, and the GitHub reference is not resolved to a repository URL in the text, so tool-version compatibility of design-file formats must be verified separately

## Evidence

- [other] Load the bundled SECIMTools example wide-format dataset (samples as columns, metabolomics features as rows with unique feature identifiers) and its design file that maps each sample to its treatment group annotation.: [evidence span withheld — anchor status: not_found]
- [abstract] Standard data formats allow arbitrary linkages between tools.: "Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs."
- [methods] The Threshold Based Flags tool identifies features below a user specified threshold in more than 50% of samples within a given group.: "The Threshold Based Flags tool identifies features below a user specified threshold in more than 50% of samples within a given group."
- [methods] The Imputation tool includes the use of the group mean or group median in place of any missing values.: "The Imputation tool includes the use of the group mean or group median in place of any missing values as well as K-nearest neighbor (KNN)"
- [methods] The files available in both public repositories can be imported into Galaxy and used in SECIMTools.: "The files available in both public repositories can be imported into Galaxy and used in SECIMTools."
