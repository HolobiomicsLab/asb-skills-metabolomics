---
name: metabolomics-blank-feature-filtering
description: Use when an untargeted metabolomics feature-intensity table contains
  blank samples alongside experimental samples and features must be flagged/removed
  based on blank-derived limits of detection before downstream statistics.
when_to_use_negative:
- No blank samples exist in the design file, so no blank-derived limit of detection
  can be computed — use Threshold Based Flags or CV Flags instead
- The goal is QC on sample quality rather than feature removal — the suite's other
  QC tools (RT Flags, Run Order Regression, Standardized Euclidean Distance) are the
  appropriate choice
- The table is raw instrument output needing peak picking / feature identification
  first; BFF operates on a preprocessed feature intensity table
edam_operation: http://edamontology.org/operation_3695
edam_topics: []
tools:
- name: SECIMTools
  role: Source implementation of the Blank Feature Filtering tool whose per-feature
    limit-of-detection logic is being reconstructed
  repo: https://github.com/SECIMTools
- name: Python
  role: Standalone language used to run the BFF reconstruction (pandas for table I/O,
    filtering, and CSV output)
  repo: https://pypi.python.org/pypi/secimtools
- name: Galaxy
  role: Alternative wrapped deployment of the BFF tool, producing Galaxy-compatible
    tsv outputs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:41c8a6b773481486ab8f3e643ab1cdb1476ac7f0095a0ef5531764e523578a29
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolomics-blank-feature-filtering

## Summary

Reimplements SECIMTools' Blank Feature Filtering (BFF) quality-control tool: it derives a per-feature limit of detection from blank-sample intensity values and removes features whose intensities in experimental samples do not exceed that blank-derived threshold.

## When to use

Use when an untargeted metabolomics feature-intensity table contains blank samples alongside experimental samples and features must be flagged/removed based on blank-derived limits of detection before downstream statistics.

## When NOT to use

- No blank samples exist in the design file, so no blank-derived limit of detection can be computed — use Threshold Based Flags or CV Flags instead
- The goal is QC on sample quality rather than feature removal — the suite's other QC tools (RT Flags, Run Order Regression, Standardized Euclidean Distance) are the appropriate choice
- The table is raw instrument output needing peak picking / feature identification first; BFF operates on a preprocessed feature intensity table

## Inputs

- Wide-format feature intensity table (samples as columns, features as rows, unique feature identifiers), e.g. the SECIMTools bundled example feature table
- Companion design file with sample annotations used to distinguish blank samples from experimental/treatment-group samples

## Outputs

- bff_filtered_features.csv: filtered wide-format feature table containing only above-threshold features
- Per-feature BFF limit-of-blank summary-statistics columns
- Per-feature 0/1 BFF flag column (0 = passed/above threshold, 1 = at or below threshold)

## How to apply

Load the wide-format feature intensity table (samples as columns, features as rows, unique feature identifiers) and its companion design file from the SECIMTools bundled test data with pandas. Use the design-file annotations to partition samples into blank samples versus experimental/treatment-group samples. For each feature, compute the limit of detection (limit of blank) from the observed blank-sample intensity values, following the BFF mechanism in which the tool 'calculates a limit of detection based upon values for a feature'. Assign a 0/1 indicator flag per feature — features above the blank threshold flagged 0 (passed), features at or below flagged 1 — and retain only features whose intensities exceed the blank-derived threshold. Write the filtered wide-format table, including the BFF summary-statistics and flag columns, to bff_filtered_features.csv in a tab/tabular format consistent with Galaxy-compatible tsv outputs.

## Related tools

- **SECIMTools** (Source implementation of the Blank Feature Filtering tool whose per-feature limit-of-detection logic is being reconstructed) — https://github.com/SECIMTools
- **Python** (Standalone language used to run the BFF reconstruction (pandas for table I/O, filtering, and CSV output)) — https://pypi.python.org/pypi/secimtools
- **Galaxy** (Alternative wrapped deployment of the BFF tool, producing Galaxy-compatible tsv outputs)

## Examples

```
import pandas as pd; feat = pd.read_csv('feature_table.csv'); design = pd.read_csv('design.tsv'); blank = feat[feat['rowID'].isin(design.columns)]; lod = blank.quantile(0.99, axis=1); feat['flag'] = (feat[labs] <= lod[:,None]).any(axis=1).astype(int); feat[feat['flag']==0].to_csv('bff_filtered_features.csv')
```

## Evaluation signals

- The reconstruction script runs on the SECIMTools bundled example input without error
- bff_filtered_features.csv exists after execution
- The output contains the per-feature BFF limit-of-blank summary fields and a 0/1 flag column
- Only features whose intensities exceed the blank-derived threshold are retained in the filtered table
- The output format is tab/tabular and consistent with Galaxy-compatible tsv outputs

## Limitations

- The article states the BFF tool 'calculates a limit of detection based upon values for a feature' but does not specify the exact limit-of-blank formula or default multiplier/threshold parameters
- Blank-versus-experimental partitioning depends entirely on the design file annotations being correctly labeled
- No SECIMTools or Galaxy version is pinned in the discussion; the GitHub reference is not resolved to a repository URL in the article text

## Evidence

- [methods] The Blank Feature Filtering (BFF) tool calculates a limit of detection based upon values for a feature: "The Blank Feature Filtering (BFF) tool calculates a limit of detection based upon values for a feature"
- [intro] New functionality includes: blank feature filtering: "New functionality includes: blank feature filtering"
- [readme] The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy.: "The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
- [abstract] Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs.: "Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs."
