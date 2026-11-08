---
name: molecular-species-submatrix-extraction
description: Use when a lipidomic intensity matrix needs to be reduced to the molecular-species
  (LMS) level before LipidOne 2.0 statistical or pathway analysis and it still contains
  missing or '0' entries that must be imputed per-row.
when_to_use_negative:
- The matrix already contains no missing or zero values — imputation would needlessly
  replace nothing or alter real measurements.
- The analysis targets the lipid class (LCL) or building block (LBB) level, where
  sub-matrix derivation follows different rules than the LMS level.
- Zeros represent genuine below-limit measurements the analyst wants to keep as such
  rather than substitute with small random values.
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipidOne 2.0
  role: web platform whose LMS sub-matrix generation this skill reconstructs; performs
    the missing/zero-to-random-value substitution automatically
  repo: https://lipidone.eu
- name: R
  role: back-end language implementing LipidOne 2.0's scripts, usable to replicate
    the imputation step
- name: LipidOne
  role: desktop Perl-based predecessor focused on building-block analysis of lipidomic
    datasets
  repo: https://github.com/matteogiulietti/LipidOne
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:0e561f95960f33cb9be841b22e967cfaae25fd8ba6e738167c9dca6adbf2e4ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# molecular-species-submatrix-extraction

## Summary

Extracts the lipid molecular species (LMS) sub-matrix from an uploaded lipidomic data matrix and imputes missing or zero intensity values by sampling uniformly between one-twentieth and one-tenth of each row's minimum valid value, as implemented in LipidOne 2.0.

## When to use

Use when a lipidomic intensity matrix needs to be reduced to the molecular-species (LMS) level before LipidOne 2.0 statistical or pathway analysis and it still contains missing or '0' entries that must be imputed per-row.

## When NOT to use

- The matrix already contains no missing or zero values — imputation would needlessly replace nothing or alter real measurements.
- The analysis targets the lipid class (LCL) or building block (LBB) level, where sub-matrix derivation follows different rules than the LMS level.
- Zeros represent genuine below-limit measurements the analyst wants to keep as such rather than substitute with small random values.

## Inputs

- lipidomic data matrix at the molecular-species (LMS) level (e.g., CSV intensity table with species rows and sample columns)
- example lipidomic datasets: ERYTHROCYTES, FUNGUS, or SCIATIC NERVE

## Outputs

- lms_submatrix.csv — completed LMS sub-matrix with no missing or zero values

## How to apply

Load the input lipidomic data matrix at the molecular-species level with pandas (e.g., one of the example datasets ERYTHROCYTES, FUNGUS, or SCIATIC NERVE shipped with the LipidOne 2.0 protocol). Retain rows of lipid molecular species and their sample intensity columns to form the LMS sub-matrix. For each row, compute the minimum among valid (non-missing, non-zero) entries, then replace every missing entry or entry equal to '0' with a uniformly drawn random number bounded by one-twentieth and one-tenth of that row's minimum. Write the completed matrix to lms_submatrix.csv; this mirrors the automatic substitution LipidOne 2.0 performs so downstream mono- and multivariate analyses are not distorted by zero placeholders.

## Related tools

- **LipidOne 2.0** (web platform whose LMS sub-matrix generation this skill reconstructs; performs the missing/zero-to-random-value substitution automatically) — https://lipidone.eu
- **R** (back-end language implementing LipidOne 2.0's scripts, usable to replicate the imputation step)
- **LipidOne** (desktop Perl-based predecessor focused on building-block analysis of lipidomic datasets) — https://github.com/matteogiulietti/LipidOne

## Examples

```
import pandas as pd; df = pd.read_csv('lms_input.csv'); row_min = df.replace(0, float('nan')).min(axis=1)
```

## Evaluation signals

- lms_submatrix.csv exists and preserves the dimensions of the input LMS block (same rows of molecular species and sample columns)
- no remaining missing values in lms_submatrix.csv
- no remaining entries equal to 0 in lms_submatrix.csv
- every imputed value lies within [row_min/20, row_min/10] where row_min is the minimum valid (non-missing, non-zero) value of that row

## Limitations

- Imputation is stochastic: values are uniformly drawn between one-twentieth and one-tenth of the row minimum, so repeated runs yield slightly different matrices.
- The rule assumes zeros and blanks are absent values, not true measurements, so datasets where 0 is meaningful are unsuitable.
- LipidOne 2.0's own deployment details (Docker, conda, source install) and software license are not stated in the article section; the desktop LipidOne predecessor is Windows-only, written in Perl.
- Example datasets (ERYTHROCYTES, FUNGUS, SCIATIC NERVE) are only accessible through the web platform rather than a persistent repository accession.

## Evidence

- [abstract] It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains).: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [intro] conducting statistical or metabolic analyses for each of these three levels requires the prior application of time- and labor-intensive calculations: [evidence span withheld — anchor status: not_found]
- [readme] LipidOne is a stand alone software entirely written in Perl and implemented for the Microsoft Windows operating system.: "LipidOne is a stand alone software entirely written in Perl and implemented for the Microsoft Windows operating system."
