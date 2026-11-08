---
name: low-variance-metabolite-flagging
description: Use when an uploaded .csv or .xlsx metabolomics table has been parsed
  (primary ID, group/factor, and metabolite concentration columns identified) and
  metabolite columns must be screened for near-constant values before imputation and
  normalization.
when_to_use_negative:
- The table has not yet been through Viime's upload-time validation for missing data,
  non-numeric concentration values, or primary ID presence/uniqueness — variance warnings
  come alongside those error checks, not before them
- The goal is formal univariate testing (e.g. Wilcoxon rank sum test or ANOVA with
  post-hoc Tukey) rather than pre-analysis quality screening of metabolite columns
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Viime
  role: Provides the upload-time validation that detects columns containing excessively
    low variance and warns that the low-variance metabolite may be omitted
- name: Pandas
  role: Backend library used for raw file parsing of the .csv/.xlsx table on which
    per-metabolite variance is computed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:96586fb3fc11cfd8659e3d69adc5db48833e71917e7a8839648959380071e1fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Viime
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# low-variance-metabolite-flagging

## Summary

A validation step in Viime's upload-time error checking that computes per-metabolite variance across samples and warns about metabolite columns with excessively low variance, flagging them as possible candidates for omission.

## When to use

Use when an uploaded .csv or .xlsx metabolomics table has been parsed (primary ID, group/factor, and metabolite concentration columns identified) and metabolite columns must be screened for near-constant values before imputation and normalization.

## When NOT to use

- The table has not yet been through Viime's upload-time validation for missing data, non-numeric concentration values, or primary ID presence/uniqueness — variance warnings come alongside those error checks, not before them
- The goal is formal univariate testing (e.g. Wilcoxon rank sum test or ANOVA with post-hoc Tukey) rather than pre-analysis quality screening of metabolite columns

## Inputs

- Uploaded metabolomics table (.csv or .xlsx) parsed with Pandas, with designated primary ID, factor/group, and metabolite concentration columns

## Outputs

- Low-variance warning entries in the structured validation report, keyed by affected metabolite column, noting the metabolite as a candidate for omission

## How to apply

Load the uploaded .csv or .xlsx table with Pandas, identifying the primary ID column, group/factor columns, and metabolite concentration columns. For each metabolite column, compute the variance of its concentration values across samples, and separately within group contexts, after handling non-numeric values that other validation checks flag as errors. Emit a warning (not a hard error) for any metabolite whose variance is excessively low; Viime surfaces such a low-variance metabolite as flagged for possible omission, as illustrated in Figure 2 of the paper. Assemble the warning together with the other detected errors and warnings — missing-data rates exceeding the default threshold, non-numeric concentration data, and missing or non-unique primary IDs — into a structured report keyed by row/column so the user can correct or exclude the flagged column in the interactive error-correction step.

## Related tools

- **Viime** (Provides the upload-time validation that detects columns containing excessively low variance and warns that the low-variance metabolite may be omitted)
- **Pandas** (Backend library used for raw file parsing of the .csv/.xlsx table on which per-metabolite variance is computed)

## Examples

```
flask run  # start Viime backend, upload the .csv/.xlsx table, and review the validation report's low-variance warnings per metabolite column
```

## Evaluation signals

- A metabolite column with essentially constant concentration values across all samples (and within each group) generates a warning in the validation report rather than being silently passed through
- The warning is classified as a warning (likely problem), not a hard error, so it does not by itself block the dataset from proceeding to imputation
- The report entry identifies the affected metabolite column so the user can omit it during Viime's error-correction step
- High-variance metabolite columns in the same table produce no low-variance warning
- The warning appears alongside the other validation outputs: threshold-exceeding missing-data rates, non-numeric concentration data, and missing/non-unique primary ID

## Limitations

- The low-variance check is a warning-level heuristic — Viime flags the metabolite as possibly omitted but does not automatically drop it; the decision is left to the user during error correction
- Variance is judged against a default threshold ('excessively low' is not user-parameterized in the article description), so borderline metabolites may be missed or over-flagged
- Non-numeric values in a concentration column corrupt the variance estimate; the non-numeric-data error check must be resolved first for a reliable variance computation

## Evidence

- [abstract] low-variance metabolite flagged for possible omission: [evidence span withheld — anchor status: not_found]
- [abstract] validation detects low-variance columns among warnings: [evidence span withheld — anchor status: not_found]
- [abstract] upload interface and column designation: "interactive drag and drop data upload interface which currently accepts .xlsx and .csv files"
- [abstract] user corrects errors after validation: "The user then is able to correct any errors, designating any column as the primary ID, masked/hidden, a factor, the group, or a metabolite concentration column"
- [readme] Pandas used for backend file parsing: "The backend leverages Pandas for raw file parsing and normalization."
