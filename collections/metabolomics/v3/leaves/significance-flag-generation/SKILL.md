---
name: significance-flag-generation
description: Use when a wide-format metabolomics feature table plus a design file
  mapping samples to treatment groups needs per-feature ANOVA contrasts reduced to
  binary significance flags for downstream reporting or filtering.
when_to_use_negative:
- The comparison is non-parametric in nature — SECIMTools offers the Kruskal-Wallis
  non-parametric test and t-test as separate basic statistical analysis methods
- Input is raw instrument data or un-preprocessed spectra rather than a feature table
  with a design file (data preprocessing and quality-control filter steps such as
  BFF or CV Flags come first in the SECIMTools workflow)
- The experimental design includes random effects or more complex factors than a single
  fixed treatment group factor
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3320
tools:
- name: SECIMTools
  role: Provides the single-factor fixed ANOVA tool whose per-feature contrast p-values
    are converted into binary significance flags
  repo: https://github.com/SECIMTools
- name: Python
  role: Implementation language used to fit the per-feature ANOVA, compute flags,
    and write anova_results.csv; SECIMTools is a set of Python applications
  repo: https://pypi.python.org/pypi/secimtools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:05c284d38b59abcc01ae1a64619f2853bf1005714b893e489f2fce986dbd2387
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# significance-flag-generation

## Summary

Derives per-feature binary significance flag columns from pairwise-group contrast p-values produced by the SECIMTools single-factor fixed ANOVA, alongside group means and F-statistics, in the anova_results.csv output table.

## When to use

Use when a wide-format metabolomics feature table plus a design file mapping samples to treatment groups needs per-feature ANOVA contrasts reduced to binary significance flags for downstream reporting or filtering.

## When NOT to use

- The comparison is non-parametric in nature — SECIMTools offers the Kruskal-Wallis non-parametric test and t-test as separate basic statistical analysis methods
- Input is raw instrument data or un-preprocessed spectra rather than a feature table with a design file (data preprocessing and quality-control filter steps such as BFF or CV Flags come first in the SECIMTools workflow)
- The experimental design includes random effects or more complex factors than a single fixed treatment group factor

## Inputs

- Wide-format metabolomics feature table (samples as columns, metabolomics features as rows with unique feature identifiers)
- Design file mapping each sample to its treatment group annotation
- Per-feature pairwise group contrast p-values from the fixed ANOVA model

## Outputs

- anova_results.csv — per-feature table of group means, ANOVA F-statistics, raw contrast p-values, and binary significance flag columns
- Optional volcano plots (log10 p-value vs difference between group means) and model diagnostic plots

## How to apply

Load the bundled SECIMTools wide-format dataset (samples as columns, features as rows with unique feature identifiers) and the design file that maps each sample to its treatment group annotation. For each feature row, fit the fully fixed single-factor ANOVA across treatment groups — the fixed-effects framework assumes normal, identically distributed errors — and calculate all pairwise contrasts. Extract the raw p-value for each group contrast and compute per-feature group means and the ANOVA F-statistic. Convert each contrast p-value into a binary significance flag column (flag set when the p-value crosses the chosen significance cutoff), then write group means, F-statistics, p-values, and flags to anova_results.csv. Optionally render the volcano plots (log base 10 p-value against the difference between group means) and model diagnostics the tool produces.

## Related tools

- **SECIMTools** (Provides the single-factor fixed ANOVA tool whose per-feature contrast p-values are converted into binary significance flags) — https://github.com/SECIMTools
- **Python** (Implementation language used to fit the per-feature ANOVA, compute flags, and write anova_results.csv; SECIMTools is a set of Python applications) — https://pypi.python.org/pypi/secimtools

## Examples

```
import pandas as pd; df = pd.read_csv("anova_results.csv"); df["flag"] = (df["p_value"] < 0.05).astype(int)
```

## Evaluation signals

- anova_results.csv contains one row per feature keyed by the unique feature identifier from the wide-format input table
- Every pairwise group contrast has both a raw p-value and a corresponding binary flag column, and each flag value matches the p-value against the significance cutoff
- Per-feature group means and the ANOVA F-statistic are populated for every feature row
- Flag columns are strictly binary (0/1) with no missing values
- Number of features in anova_results.csv equals the number of feature rows in the input wide-format dataset

## Limitations

- The fixed-effects ANOVA relies on the standard assumptions of normal and identically distributed errors; violations are not handled by this flag-generation step
- Flags are derived from raw p-values, not adjusted/corrected p-values — multiple-testing behavior across features is not addressed in the described tool output
- No version number for SECIMTools or Galaxy is given in the discussion; the GitHub reference is not resolved to a repository URL in the text

## Evidence

- [abstract] basic statistical analysis methods: "basic statistical analysis methods (partial least squares - discriminant analysis, analysis of variance, t-test, Kruskal-Wallis non-parametric test)"
- [readme] SECIMTools ANOVA in README: "basic statistical analysis methods (partial least squares - discriminant analysis, analysis of variance)"
- [methods] Per-feature ANOVA tool context: "SECIMTools is open source, the code, is available on GitHub using the MIT license"
