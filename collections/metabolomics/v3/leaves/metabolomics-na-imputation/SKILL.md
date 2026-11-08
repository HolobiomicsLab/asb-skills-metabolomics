---
name: metabolomics-na-imputation
description: Use when a tab-delimited feature abundance table (e.g. from MS-DIAL,
  such as margheRita's 'Urine' dataset) contains NA metabolite abundances that must
  be filled with low, plausible values before downstream statistical analysis.
when_to_use_negative:
- 'The dataset already has no missing values — margheRita itself skips the step and
  reports ''No NAs: imputation not performed.'''
- A model-based imputation is required (e.g. random forest imputation as recommended
  in the notame workflow) rather than a simple random-below-minimum replacement
- Input is raw instrument data or an MS-DIAL peak-picking table not yet reduced to
  the feature abundance matrix that filtering() operates on
edam_operation: http://edamontology.org/operation_3914
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: margheRita
  role: Provides the imputation(mRList, seed, a, b) function executed as part of the
    filtering() pre-processing chain, with a = 10% and b = 25% of the feature minimum
    by default
  repo: https://github.com/emosca-cnr/margheRita
- name: R
  role: Runtime environment in which the margheRita filtering()/imputation() chain
    is executed on the tab-delimited feature table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.06.20.599545
    title: 'MargheRita: streamlining MS-DIAL output analysis and metabolite identification
      in R'
  source_content_hash: sha256:dfad3604ee3df247a2b1a9e52d3f1fa4268b3efe98db00e4d67580c504da7107
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emosca-cnr/margheRita
  tools:
  - R
  - margheRita
  edam_operation: http://edamontology.org/operation_3914
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# metabolomics-na-imputation

## Summary

Imputes missing (NA) metabolite abundance values in MS-DIAL-derived LC-MS/MS feature tables by replacing each NA with a seeded random number drawn between 10% and 25% of the corresponding feature's minimum observed value, as implemented in the margheRita filtering() pre-processing chain.

## When to use

Use when a tab-delimited feature abundance table (e.g. from MS-DIAL, such as margheRita's 'Urine' dataset) contains NA metabolite abundances that must be filled with low, plausible values before downstream statistical analysis.

## When NOT to use

- The dataset already has no missing values — margheRita itself skips the step and reports 'No NAs: imputation not performed.'
- A model-based imputation is required (e.g. random forest imputation as recommended in the notame workflow) rather than a simple random-below-minimum replacement
- Input is raw instrument data or an MS-DIAL peak-picking table not yet reduced to the feature abundance matrix that filtering() operates on

## Inputs

- Tab-delimited feature abundance table from MS-DIAL output (feature identifiers, m/z, retention times, MS/MS spectra, abundances), e.g. the margheRita 'Urine' dataset (10.5281/zenodo.11243781)
- mRList object holding the feature abundance matrix within margheRita's pre-processing chain

## Outputs

- Imputed feature abundance matrix with no remaining NA values (e.g. written to imputed_feature_abundances.csv)
- Console message 'No NAs: imputation not performed.' when the input contains no NA values

## How to apply

Load the tab-delimited feature abundance table into R via margheRita and detect NA values per feature. For each feature containing NAs, compute its minimum observed abundance and draw a random number uniformly between a = 10% and b = 25% of that minimum, using a fixed seed (the imputation() signature is imputation(mRList, seed, a, b)) so results are reproducible. Values are drawn below the detection level because missing signals in untargeted metabolomics typically reflect absence or below-threshold abundance, not high abundance. If the table contains no NAs, skip the step and emit the message 'No NAs: imputation not performed.' as margheRita's filtering() chain does. Write the imputed matrix and verify no NA values remain before proceeding to normalization (e.g. probabilistic quotient normalization) and statistical testing.

## Related tools

- **margheRita** (Provides the imputation(mRList, seed, a, b) function executed as part of the filtering() pre-processing chain, with a = 10% and b = 25% of the feature minimum by default) — https://github.com/emosca-cnr/margheRita
- **R** (Runtime environment in which the margheRita filtering()/imputation() chain is executed on the tab-delimited feature table)

## Examples

```
Rscript -e 'library(margheRita); imputed <- imputation(mRList, seed = 123, a = 0.10, b = 0.25)'
```

## Evaluation signals

- Output abundance matrix contains zero NA values after imputation
- Every imputed value falls within [0.10 × feature minimum, 0.25 × feature minimum] for its corresponding feature
- Re-running with the same seed reproduces the identical imputed matrix
- When the input has no NAs, the expected message 'No NAs: imputation not performed.' is emitted and the data pass through unchanged
- Feature and sample counts are unchanged by imputation (e.g. no features or samples are dropped, unlike the NA-based filter_NA step)

## Limitations

- Reproducibility requires an explicit fixed seed; different seeds yield different imputed values
- Imputation is skipped entirely when the dataset contains no NAs, so downstream results are unchanged rather than re-randomized
- The 10%–25%-of-minimum range is a fixed default; values below the detection limit assumption may not suit all instrument workflows

## Evidence

- [intro] the imputation is performed replacing NA values with a random number, calculated between 10%-25% of the minimum value of the feature.: "the imputation is performed replacing NA values with a random number, calculated between 10%-25% of the minimum value of the feature."
- [results] Features without appropriate m/z values: 56 
No NAs: imputation not performed.: "Features without appropriate m/z values: 56 
No NAs: imputation not performed."
- [readme] a series of pre-processing functions (quality control, filtering and normalization): "a series of pre-processing functions (quality control, filtering and normalization) with a particular focus on methods specifically recommended for metabolomic profiles"
