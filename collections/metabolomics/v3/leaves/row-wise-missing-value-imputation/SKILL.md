---
name: row-wise-missing-value-imputation
description: Use when a lipidomic intensity matrix at the molecular-species (LMS)
  level contains absent cells or zeros that must be filled before downstream statistics,
  and row-relative low-value imputation is acceptable.
when_to_use_negative:
- The input matrix is at the lipid class (LCL) or building block (LBB) structural
  level rather than the molecular-species (LMS) level
- The matrix already contains no missing or zero values, so imputation would only
  add artificial noise
- A deterministic, seed-controlled missing-value method is required and the non-deterministic
  random draw is unacceptable
tools:
- name: LipidOne 2.0
  role: Web tool whose LMS sub-matrix generation implements this row-wise missing/zero
    imputation rule
  repo: https://lipidone.eu
- name: R
  role: Back-end language in which LipidOne 2.0's data-processing scripts (including
    imputation) are coded
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:611f8c0995a59a9183b415e385fc40581f991764c7d367139c674f3c5f2aa8d2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne 2.0
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# row-wise-missing-value-imputation

## Summary

Reconstructs a complete lipid molecular species (LMS) sub-matrix from an uploaded lipidomic data matrix by imputing every missing or '0' entry with a random number drawn between one-twentieth and one-tenth of the row's minimum valid value, as done by LipidOne 2.0.

## When to use

Use when a lipidomic intensity matrix at the molecular-species (LMS) level contains absent cells or zeros that must be filled before downstream statistics, and row-relative low-value imputation is acceptable.

## When NOT to use

- The input matrix is at the lipid class (LCL) or building block (LBB) structural level rather than the molecular-species (LMS) level
- The matrix already contains no missing or zero values, so imputation would only add artificial noise
- A deterministic, seed-controlled missing-value method is required and the non-deterministic random draw is unacceptable

## Inputs

- Lipidomic data matrix at the molecular-species (LMS) level (rows = lipid molecular species, columns = sample intensity values)
- Example lipidomic datasets shipped with the protocol (e.g., ERYTHROCYTES, FUNGUS, SCIATIC NERVE)

## Outputs

- Completed LMS sub-matrix with no missing or zero values (e.g., lms_submatrix.csv)

## How to apply

Load the input lipidomic data matrix at the molecular-species level with pandas, retaining rows of lipid molecular species and their sample intensity columns. For each row, compute the minimum value among the valid (non-missing, non-zero) entries, then replace every missing entry or entry equal to '0' with a uniformly drawn random number bounded by one-twentieth and one-tenth of that row's minimum. The rationale is that LipidOne 2.0 performs this substitution automatically so that the LMS sub-matrix contains no zero or missing values before mono- and multivariate analysis. Write the completed sub-matrix (e.g., to lms_submatrix.csv) and verify it preserves the dimensions of the input LMS block, contains no remaining missing or zero values, and that every imputed value lies within [row_min/20, row_min/10].

## Related tools

- **LipidOne 2.0** (Web tool whose LMS sub-matrix generation implements this row-wise missing/zero imputation rule) — https://lipidone.eu
- **R** (Back-end language in which LipidOne 2.0's data-processing scripts (including imputation) are coded)

## Examples

```
import pandas as pd, numpy as np; df = pd.read_csv('lms_input.csv', index_col=0); [df.loc[i, row.isna() | (row == 0)] = np.random.uniform(row[~(row.isna() | (row == 0))].min()/20, row[~(row.isna() | (row == 0))].min()/10) for i, row in df.iterrows()]; df.to_csv('lms_submatrix.csv')
```

## Evaluation signals

- lms_submatrix.csv exists and preserves the dimensions of the input LMS block
- The output contains no remaining missing (NaN) values
- The output contains no remaining zero values
- Every imputed value lies within [row_min/20, row_min/10] for its row
- Non-imputed (original non-missing, non-zero) entries are unchanged from the input matrix

## Limitations

- Imputed values are randomly drawn within [row_min/20, row_min/10], so results are non-deterministic unless a random seed is fixed
- Zeros are treated as missing rather than as true biological measurements, which may distort rows where zero is a genuine low signal
- The LipidOne desktop source in the GitHub repository is a Perl/Windows standalone predecessor; LipidOne 2.0 itself is accessed as a web tool, and example datasets are only accessible through the web platform rather than a persistent repository accession

## Evidence

- [other] The LMS sub-matrix is generated from the input data by automatically replacing numeric values that are absent or set to "0" with a random number between one-tenth and one-twentieth of the minimum value in the same row.: [evidence span withheld — anchor status: not_found]
- [abstract] It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains).: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [readme] LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset.: "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."
