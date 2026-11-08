---
name: wide-format-feature-table-processing
description: Use when the input is a wide-format feature intensity table (unique feature
  identifiers as rows, samples as columns) plus a companion design file, and the goal
  is blank-based quality-control filtering of metabolomic features.
when_to_use_negative:
- The data are still raw instrument acquisitions needing peak picking/feature identification,
  which the article treats as a platform-specific step upstream of SECIMTools feature-table
  QC
- The goal is a QC check other than blank-based detection (e.g. Threshold Based Flags,
  CV Flags, Run Order Regression, retention time flags), which use group statistics
  or run-order rather than blank-sample intensities
- There are no blank samples in the design file, so no per-feature limit of detection
  from blanks can be computed
tools:
- name: SECIMTools
  role: Source suite providing the Blank Feature Filtering (BFF) QC tool and the wide-format
    tabular data conventions the filtered table must follow; usable standalone or
    via Galaxy.
  repo: https://github.com/SECIMTools
- name: Galaxy
  role: 'Execution/compatibility target: SECIMTools is wrapped for use in Galaxy,
    and the filtered output keeps Galaxy-compatible tsv/tabular formatting.'
- name: Python
  role: Implementation language used to load the example wide-format feature table
    and design file with pandas and compute the BFF flags.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:1b0e2bc9ed3da3d0c3bc4e00b6cda177904148d727c85251469dfb7cbce3907d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# wide-format-feature-table-processing

## Summary

Loading, annotating, and quality-filtering a wide-format metabolomic feature intensity table (samples as columns, features as rows) using SECIMTools-style QC steps such as Blank Feature Filtering (BFF), which computes a per-feature limit of detection from blank samples.

## When to use

Use when the input is a wide-format feature intensity table (unique feature identifiers as rows, samples as columns) plus a companion design file, and the goal is blank-based quality-control filtering of metabolomic features.

## When NOT to use

- The data are still raw instrument acquisitions needing peak picking/feature identification, which the article treats as a platform-specific step upstream of SECIMTools feature-table QC
- The goal is a QC check other than blank-based detection (e.g. Threshold Based Flags, CV Flags, Run Order Regression, retention time flags), which use group statistics or run-order rather than blank-sample intensities
- There are no blank samples in the design file, so no per-feature limit of detection from blanks can be computed

## Inputs

- Wide-format feature intensity table (features as rows with unique feature identifiers, samples as columns), e.g. the SECIMTools bundled example test-data file
- Companion design/annotation file mapping each sample to blank vs. experimental/treatment-group categories

## Outputs

- Filtered wide-format feature table written to bff_filtered_features.csv in Galaxy-compatible tab/tsv format
- Per-feature BFF limit-of-blank summary-statistics columns
- Per-feature 0/1 BFF flag column (1 = at or below blank threshold, 0 = passed)

## How to apply

Load the wide-format feature table and its companion design file with pandas, using the design-file annotations to partition samples into blank samples and experimental/treatment-group samples. For each feature, compute a limit of detection (limit of blank) from the observed blank-sample intensity values, as the SECIMTools BFF tool does when it 'calculates a limit of detection based upon values for a feature'. Assign a 0/1 indicator flag per feature — features above the blank-derived threshold flagged 0 (passed), features at or below flagged 1 — and retain only features whose intensities exceed the blank-derived threshold. Write the filtered wide-format table, including the BFF summary-statistics and flag columns, to a tabular output such as bff_filtered_features.csv, keeping the Galaxy-compatible tsv/tabular conventions SECIMTools uses so the file can chain into downstream tools via 'standard data formats and a set of utilities allow[ing] arbitrary linkages between tools'.

## Related tools

- **SECIMTools** (Source suite providing the Blank Feature Filtering (BFF) QC tool and the wide-format tabular data conventions the filtered table must follow; usable standalone or via Galaxy.) — https://github.com/SECIMTools
- **Galaxy** (Execution/compatibility target: SECIMTools is wrapped for use in Galaxy, and the filtered output keeps Galaxy-compatible tsv/tabular formatting.)
- **Python** (Implementation language used to load the example wide-format feature table and design file with pandas and compute the BFF flags.)

## Examples

```
import pandas as pd; df = pd.read_csv('features.csv', index_col=0); design = pd.read_csv('design.tsv', sep='\t')
```

## Evaluation signals

- The reconstruction script runs on the bundled SECIMTools example input and bff_filtered_features.csv exists
- Output CSV contains the per-feature BFF limit-of-blank summary fields and a 0/1 flag column
- Only features whose intensities exceed the blank-derived threshold are retained; rows flagged 1 are absent from the filtered table
- Retained feature count is <= input feature count, and unique feature identifiers are preserved row-by-row
- Output is a tab/tsv file consistent with Galaxy-compatible tabular outputs

## Limitations

- The BFF decision depends entirely on the user-supplied design file correctly flagging which samples are blanks; mislabeled blanks shift the computed limit of detection
- Features at or below the blank-derived limit of detection are flagged 1 and removed, so genuinely low-abundance endogenous features can be lost along with background contamination
- The article gives no SECIMTools or Galaxy version numbers, so exact default thresholds and output columns cannot be pinned down from the text alone

## Evidence

- [methods] BFF mechanism: "The Blank Feature Filtering (BFF) tool calculates a limit of detection based upon values for a feature"
- [abstract] workflow linkage: "Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs."
- [intro] new QC functionality: "New functionality includes: blank feature filtering"
- [readme] standalone/Galaxy modes: "The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
