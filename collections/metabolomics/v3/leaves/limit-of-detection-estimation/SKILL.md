---
name: limit-of-detection-estimation
description: Use when a wide-format feature intensity table includes blank samples
  in its companion design file and features must be flagged/filtered against a blank-derived
  limit of detection before downstream statistics.
when_to_use_negative:
- The dataset contains no blank samples in the design file, so no blank-based limit
  of detection can be computed; a threshold-based criterion such as the Threshold
  Based Flags tool (features below a user-specified threshold in >50% of samples within
  a group) is the alternative.
- The feature table has already been blank-filtered and the goal is downstream statistical
  analysis (ANOVA, t-test, Kruskal-Wallis, PLS-DA) or visualization (PCA, hierarchical
  cluster heatmap).
- The goal is feature annotation or peak picking rather than post-acquisition quality
  control; SECIMTools complements tools like Galaxy-M (peak-picking/feature identification)
  and Workflow4Metabolomics (feature annotation).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: SECIMTools
  role: Provides the Blank Feature Filtering (BFF) tool that computes the per-feature
    limit of detection from blank values, assigns 0/1 flags, and writes the filtered
    wide-format feature table; installable as a secimtools pypi package.
  repo: https://github.com/SECIMTools
- name: Galaxy
  role: Web platform through which SECIMTools, including the BFF quality-control filter,
    can be run as wrapped tools in integrated metabolomics workflows producing Galaxy-compatible
    tsv outputs.
- name: Python
  role: Language in which SECIMTools is implemented and in which the standalone reconstruction
    (pandas loading of the feature table and design file, per-feature limit-of-blank
    computation, tsv output) is written.
  repo: https://pypi.python.org/pypi/secimtools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:52dbb6ecf01f540c7618c269bd788cabb6539d078ee4683052eb6c021730700d
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
yes_rate: 0.8461538461538461
n_claims: 13
---

# limit-of-detection-estimation

## Summary

Estimates a per-feature limit of detection (limit of blank) from blank-sample intensities, as implemented in SECIMTools' Blank Feature Filtering (BFF) tool, and uses it to flag and remove metabolomic features at or below blank-level signal.

## When to use

Use when a wide-format feature intensity table includes blank samples in its companion design file and features must be flagged/filtered against a blank-derived limit of detection before downstream statistics.

## When NOT to use

- The dataset contains no blank samples in the design file, so no blank-based limit of detection can be computed; a threshold-based criterion such as the Threshold Based Flags tool (features below a user-specified threshold in >50% of samples within a group) is the alternative.
- The feature table has already been blank-filtered and the goal is downstream statistical analysis (ANOVA, t-test, Kruskal-Wallis, PLS-DA) or visualization (PCA, hierarchical cluster heatmap).
- The goal is feature annotation or peak picking rather than post-acquisition quality control; SECIMTools complements tools like Galaxy-M (peak-picking/feature identification) and Workflow4Metabolomics (feature annotation).

## Inputs

- Wide-format feature intensity table (rows = features with unique feature identifiers, columns = samples), e.g. tab-delimited tsv
- Companion design file with sample annotations flagging which samples are blanks versus experimental/treatment groups

## Outputs

- Filtered wide-format feature table (bff_filtered_features.csv) containing only features above the blank-derived limit of detection
- Per-feature BFF summary-statistics columns (blank-based limit of detection)
- Per-feature 0/1 indicator flag column (0 = passed/above threshold, 1 = at or below blank threshold)

## How to apply

Load the wide-format feature intensity table (samples as columns, features as rows, unique feature identifiers) together with its companion design file using pandas from SECIMTools' bundled test data. Use the design file annotations to partition samples into blank samples versus experimental/treatment-group samples. For each feature, compute the limit of detection (limit of blank) from the observed blank-sample intensity values, following the BFF mechanism in which the tool 'calculates a limit of detection based upon values for a feature'. Assign a 0/1 flag per feature (features above the blank threshold pass with flag 0; features at or below the threshold are flagged 1), retain only above-threshold features, and write the filtered wide-format table—including the BFF summary-statistics and flag columns—to bff_filtered_features.csv in a tab/tabular, Galaxy-compatible tsv format. This step sits in the data preprocessing phase of the metabolomics workflow, before imputation, normalization/re-scaling, or statistical analysis.

## Related tools

- **SECIMTools** (Provides the Blank Feature Filtering (BFF) tool that computes the per-feature limit of detection from blank values, assigns 0/1 flags, and writes the filtered wide-format feature table; installable as a secimtools pypi package.) — https://github.com/SECIMTools
- **Galaxy** (Web platform through which SECIMTools, including the BFF quality-control filter, can be run as wrapped tools in integrated metabolomics workflows producing Galaxy-compatible tsv outputs.)
- **Python** (Language in which SECIMTools is implemented and in which the standalone reconstruction (pandas loading of the feature table and design file, per-feature limit-of-blank computation, tsv output) is written.) — https://pypi.python.org/pypi/secimtools

## Examples

```
import pandas as pd; df = pd.read_csv('feature_intensity_table.tsv', sep='\t'); df.to_csv('bff_filtered_features.csv', sep='\t', index=False)
```

## Evaluation signals

- The output file bff_filtered_features.csv exists after running the reconstruction on the bundled example input.
- The output CSV contains per-feature BFF limit-of-blank summary-statistics fields and a 0/1 flag column.
- Every retained feature has an intensity strictly exceeding its blank-derived limit-of-detection threshold; features at or below the threshold are flagged 1 and absent from the filtered table.
- The output remains wide-format (samples as columns, features as rows with unique feature identifiers) in a tab/tabular, Galaxy-compatible tsv layout.
- The blank versus experimental sample partition used for the limit-of-blank computation matches the design file annotations exactly.

## Limitations

- The blank-based limit of detection depends entirely on the observed blank-sample values for each feature, so datasets lacking blank samples cannot be filtered with BFF; the article's alternative, Threshold Based Flags, requires a user-specified threshold and >50% of samples in a group instead.
- The article does not state a specific numeric blank-threshold multiplier or version number for SECIMTools; the discussion leaves the GitHub reference [27] unresolved to a repository URL in the text.
- BFF is one of several independent quality-control flag tools (RT flags, CV flags, Run Order Regression, Magnitude Difference Flags); it addresses blank-level signal only and does not handle run-order effects or within-group variability.

## Evidence

- [methods] The Blank Feature Filtering (BFF) tool calculates a limit of detection based upon values for a feature: "The Blank Feature Filtering (BFF) tool calculates a limit of detection based upon values for a feature"
- [intro] New functionality includes: blank feature filtering: "New functionality includes: blank feature filtering"
- [readme] SECIMTools are available as a secimtools pypi package. Project has also been packaged for bioconda and Galaxy Genomics Framework.: "SECIMTools are available as a secimtools pypi package. Project has also been packaged for bioconda and Galaxy Genomics Framework."
- [methods] The Threshold Based Flags tool identifies features below a user specified threshold in more than 50% of samples within a given group.: "The Threshold Based Flags tool identifies features below a user specified threshold in more than 50% of samples within a given group."
- [abstract] Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs.: "Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs."
