---
name: qc-flag-based-feature-filtering
description: Use when a wide-format metabolomics feature intensity table (features
  as rows, samples as columns) includes blank samples, and features must be flagged
  and filtered against a blank-derived limit of detection.
when_to_use_negative:
- The input has no blank samples (or no design-file annotations distinguishing blanks),
  so no blank-based limit of detection can be computed — use the Threshold Based Flags
  tool instead, which identifies features below a user-specified threshold in more
  than 50% of samples within a given group.
- The goal is to evaluate features by retention-time variability, run order, or coefficient
  of variation rather than blank contamination — use the RT Flags, Run Order Regression,
  or CV Flags tools instead.
- The data are raw instrument files (e.g. pre-peak-picking) rather than a preprocessed
  wide-format feature table, since BFF starts from feature identification output.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: SECIMTools
  role: Provides the Blank Feature Filtering (BFF) tool that computes a per-feature
    limit of detection from blank values and emits 0/1 QC flags; installable as a
    python package or run standalone
  repo: https://github.com/SECIMTools
- name: Galaxy
  role: Web platform in which SECIMTools tools (including BFF) are wrapped for integrated
    metabolomics QC workflows
- name: Python
  role: Language of the SECIMTools applications; used with pandas to load the wide-format
    feature table and design file for BFF reconstruction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:7b9f4572a1d4d1cde367d4c49554c0bd1d543d8c029816c67f58f3f0f8a32639
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# qc-flag-based-feature-filtering

## Summary

The Blank Feature Filtering (BFF) quality-control tool from SECIMTools computes a per-feature limit of detection from blank-sample intensity values and removes features that do not exceed that blank-derived threshold in a wide-format metabolomics feature table.

## When to use

Use when a wide-format metabolomics feature intensity table (features as rows, samples as columns) includes blank samples, and features must be flagged and filtered against a blank-derived limit of detection.

## When NOT to use

- The input has no blank samples (or no design-file annotations distinguishing blanks), so no blank-based limit of detection can be computed — use the Threshold Based Flags tool instead, which identifies features below a user-specified threshold in more than 50% of samples within a given group.
- The goal is to evaluate features by retention-time variability, run order, or coefficient of variation rather than blank contamination — use the RT Flags, Run Order Regression, or CV Flags tools instead.
- The data are raw instrument files (e.g. pre-peak-picking) rather than a preprocessed wide-format feature table, since BFF starts from feature identification output.

## Inputs

- Wide-format feature intensity table (features as rows with unique feature identifiers, samples as columns)
- Design file with per-sample annotations/flags distinguishing blank samples from experimental/treatment-group samples

## Outputs

- Filtered wide-format feature table containing only above-threshold features, plus per-feature BFF limit-of-detection summary-statistics columns and a 0/1 flag column
- Galaxy-compatible tab/tsv output file (e.g. bff_filtered_features.csv)

## How to apply

Load the wide-format feature intensity table and its companion design file (e.g. with pandas from SECIMTools' bundled test data), and use the design-file annotations to partition samples into blank samples versus experimental/treatment-group samples. For each feature, compute the limit of detection (limit of blank) from the observed blank-sample intensity values, as the BFF tool 'calculates a limit of detection based upon values for a feature'. Assign a 0/1 indicator flag per feature — features above the blank threshold flagged 0 (passed), features at or below flagged 1 — and retain only features whose intensities exceed the blank-derived threshold. Write the filtered wide-format table, including the BFF summary-statistics and 0/1 flag columns, as a Galaxy-compatible tab/tsv output (e.g. bff_filtered_features.csv). The rationale is that QC-flag tools in SECIMTools operate on the standard feature-table format, and standard data formats allow arbitrary linkages between tools in Galaxy workflows.

## Related tools

- **SECIMTools** (Provides the Blank Feature Filtering (BFF) tool that computes a per-feature limit of detection from blank values and emits 0/1 QC flags; installable as a python package or run standalone) — https://github.com/SECIMTools
- **Galaxy** (Web platform in which SECIMTools tools (including BFF) are wrapped for integrated metabolomics QC workflows)
- **Python** (Language of the SECIMTools applications; used with pandas to load the wide-format feature table and design file for BFF reconstruction)

## Examples

```
pip install secimtools
```

## Evaluation signals

- The output CSV exists and is parseable in tab/tsar format consistent with Galaxy-compatible tsv outputs.
- The output contains per-feature BFF limit-of-blank summary-statistics fields and a 0/1 flag column (0 = passed, 1 = flagged).
- Every retained feature's intensity exceeds its blank-derived limit-of-detection threshold; no at-or-below-threshold feature rows remain.
- The sample partition into blank vs. treatment-group samples matches the companion design file annotations.
- The filtered table preserves the original unique feature identifiers and wide format (features as rows, samples as columns) of the input.

## Limitations

- The exact BFF limit-of-detection formula/parameters (beyond 'calculates a limit of detection based upon values for a feature') are not specified in the article text, so reconstruction relies on the bundled example data.
- The article's discussion gives no SECIMTools or Galaxy version number, and the GitHub reference is not resolved to a repository URL in the text, so tool-version reproducibility is unverified.
- Blank-based filtering requires a design file that correctly flags blank samples; mis-annotated samples would corrupt the limit-of-detection estimate.

## Evidence

- [methods] BFF computes limit of detection from blank values: "The Blank Feature Filtering (BFF) tool calculates a limit of detection based upon values for a feature"
- [intro] Blank feature filtering is new SECIMTools functionality: "New functionality includes: blank feature filtering"
- [readme] SECIMTools runs standalone or via Galaxy: "SECIMTools project aims to develop a suite of tools for processing of metabolomics data, which can be run in a standalone mode or via Galaxy Genomics Framework."
- [abstract] Standard data formats enable workflow linkage between SECIMTools tools: "Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs."
