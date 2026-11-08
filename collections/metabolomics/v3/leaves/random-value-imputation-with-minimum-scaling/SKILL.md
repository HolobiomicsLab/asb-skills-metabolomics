---
name: random-value-imputation-with-minimum-scaling
description: Use when an MS-DIAL-derived feature abundance table in an untargeted
  LC-MS/MS metabolomics workflow contains NA values that must be filled before normalization
  or statistical testing, and low-abundance-proportional imputation is desired.
when_to_use_negative:
- 'The feature abundance table contains no NA values — margheRita skips imputation
  and reports ''No NAs: imputation not performed.'''
- Missingness is extensive or non-random and a model-based strategy is preferred (e.g.
  notame recommends random forest imputation for missing values)
- The input is raw instrument data or an already-normalized matrix rather than a preprocessed
  MS-DIAL feature abundance table feeding the margheRita filtering() chain
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: margheRita
  role: Provides the imputation(mRList, seed, a, b) function invoked within its filtering()
    pre-processing chain to replace NA abundances
  repo: https://github.com/emosca-cnr/margheRita
- name: R
  role: Execution environment in which the margheRita imputation step is run on the
    abundance table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.06.20.599545
    title: 'MargheRita: streamlining MS-DIAL output analysis and metabolite identification
      in R'
  source_content_hash: sha256:4d8fa44a15aa9c775c6d16ec06bf6f194884a5e7bcfe5a3285830218fc3cb56f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emosca-cnr/margheRita
  tools:
  - R
  - margheRita
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Random-value imputation with minimum scaling

## Summary

Imputes missing (NA) metabolite abundances in an LC-MS/MS feature table by drawing a seeded random number between 10% (a) and 25% (b) of the minimum observed value of the corresponding feature, as implemented in the margheRita filtering() pre-processing chain.

## When to use

Use when an MS-DIAL-derived feature abundance table in an untargeted LC-MS/MS metabolomics workflow contains NA values that must be filled before normalization or statistical testing, and low-abundance-proportional imputation is desired.

## When NOT to use

- The feature abundance table contains no NA values — margheRita skips imputation and reports 'No NAs: imputation not performed.'
- Missingness is extensive or non-random and a model-based strategy is preferred (e.g. notame recommends random forest imputation for missing values)
- The input is raw instrument data or an already-normalized matrix rather than a preprocessed MS-DIAL feature abundance table feeding the margheRita filtering() chain

## Inputs

- Tab-delimited feature abundance table from MS-DIAL (feature identifiers, m/z, retention times, MS/MS spectra, abundances), e.g. the margheRita 'Urine' dataset (10.5281/zenodo.11243781)
- mRList object holding the feature abundance matrix
- Parameters: seed (random seed for reproducibility), a = 0.10 (lower fraction of feature minimum), b = 0.25 (upper fraction of feature minimum)

## Outputs

- Imputed feature abundance matrix with no remaining NA values
- Console message 'No NAs: imputation not performed.' when the input table contains no NA values

## How to apply

Load the tab-delimited feature abundance table into R as a margheRita mRList object and detect NA values per feature. For each feature containing NAs, compute its minimum observed abundance and replace each NA with a random number drawn uniformly between 10% (a) and 25% (b) of that minimum, passing a fixed seed to imputation(mRList, seed, a, b) for reproducibility. If the table contains no NA values, skip the step entirely and report the message 'No NAs: imputation not performed.', matching margheRita's filtering() chain behaviour. The rationale is to substitute missing signals with values proportionally below the lowest observed abundance of the feature, keeping imputed intensities consistent with low-abundance metabolite detection rather than introducing arbitrary mid-range values.

## Related tools

- **margheRita** (Provides the imputation(mRList, seed, a, b) function invoked within its filtering() pre-processing chain to replace NA abundances) — https://github.com/emosca-cnr/margheRita
- **R** (Execution environment in which the margheRita imputation step is run on the abundance table)

## Examples

```
imputed <- imputation(mRList, seed = 123, a = 0.10, b = 0.25)
```

## Evaluation signals

- The output abundance matrix contains zero NA values after imputation
- Every imputed value lies within [0.10 × feature minimum, 0.25 × feature minimum] for its corresponding feature
- Re-running with the same seed reproduces identical imputed values (deterministic seeded draws)
- When the input has no NAs, the exact message 'No NAs: imputation not performed.' is emitted and no values are altered

## Limitations

- The 10%–25%-of-minimum range is a fixed default; values below the feature minimum are fabricated and may bias downstream statistics if missingness is not low-abundance-driven
- Random draws depend on the seed; without a fixed seed the imputed matrix is not reproducible
- Imputation is skipped entirely when no NAs are present, so the step provides no corrective action for other data-quality issues (e.g. features failing m/z or CV filtering)

## Evidence

- [intro] Imputation replaces NA abundances with a seeded random value scaled to the feature minimum: "the imputation is performed replacing NA values with a random number, calculated between 10%-25% of the minimum value of the feature."
- [intro] Imputation is skipped with a defined message when no NAs are present: "Features without appropriate m/z values: 56 
No NAs: imputation not performed."
- [intro] Imputation is part of the margheRita pre-processing chain alongside quality control, filtering and normalization: "a series of pre-processing functions (quality control, filtering and normalization) with a particular focus on methods specifically recommended for metabolomic profiles"
