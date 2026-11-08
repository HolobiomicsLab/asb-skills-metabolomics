---
name: technical-replicate-collapsing
description: Use when an LC-MS/MS feature table contains repeated technical replicate
  injections (annotated via 'technical_rep' in the sample annotation file) and downstream
  per-feature tests such as ANOVA must be run over biological replicates only.
when_to_use_negative:
- The feature table already contains one row per biological replicate (no technical_rep
  column or no repeated injections), so collapsing would drop real samples
- The goal is QC or drift assessment at injection level, where per-technical-replicate
  measurements are the units of interest
- Input is not margheRita/mRList-format MS-DIAL output with the required id, injection_order,
  batch, class, biological_rep, technical_rep annotation columns
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: margheRita
  role: Provides the collapse_tech_rep function that merges technical replicate injections
    over samples sharing the same concatenation of 'class' and 'biological_rep' values
    within the mRList object
  repo: https://github.com/emosca-cnr/margheRita
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.06.20.599545
    title: 'MargheRita: streamlining MS-DIAL output analysis and metabolite identification
      in R'
  source_content_hash: sha256:6c8c6c103f6782212e56f24b00756c9f4937251524da2c6be139c871ba46932b
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# technical-replicate-collapsing

## Summary

Collapses technical replicate injections into single biological-replicate samples in an mRList object using margheRita's collapse_tech_rep function, grouping all samples that share the same concatenation of 'class' and 'biological_rep' values before dataset-wide statistical testing.

## When to use

Use when an LC-MS/MS feature table contains repeated technical replicate injections (annotated via 'technical_rep' in the sample annotation file) and downstream per-feature tests such as ANOVA must be run over biological replicates only.

## When NOT to use

- The feature table already contains one row per biological replicate (no technical_rep column or no repeated injections), so collapsing would drop real samples
- The goal is QC or drift assessment at injection level, where per-technical-replicate measurements are the units of interest
- Input is not margheRita/mRList-format MS-DIAL output with the required id, injection_order, batch, class, biological_rep, technical_rep annotation columns

## Inputs

- mRList object built from a tab-delimited MS-DIAL feature data file (feature identifiers, m/z, retention times, MS/MS spectra, abundances)
- Tab-delimited sample annotation file with columns id, injection_order, batch, class, biological_rep, technical_rep

## Outputs

- Collapsed mRList object in which each sample corresponds to one unique class + biological_rep combination
- Sample annotation for the collapsed object retaining class and biological_rep groupings, ready for univariate() testing

## How to apply

Load the MS-DIAL-derived feature data file (feature identifiers, m/z, retention times, MS/MS spectra, abundances) and the tab-delimited sample annotation file with columns id, injection_order, batch, class, biological_rep, technical_rep into an mRList with margheRita. Then call collapse_tech_rep on the mRList: margheRita defines the collapse of technical replicates over all samples that share the same concatenation of 'class' and 'biological_rep' values, so technical replicate injections belonging to the same biological replicate within the same sample class are merged into one sample. This step must precede running univariate() with test_method = 'anova' over the class factor levels, so that test statistics and q-values per feature are computed across independent biological units rather than correlated replicate injections. Validate the collapse by checking that the number of samples in the collapsed mRList equals the number of unique class × biological_rep combinations in the original annotation.

## Related tools

- **margheRita** (Provides the collapse_tech_rep function that merges technical replicate injections over samples sharing the same concatenation of 'class' and 'biological_rep' values within the mRList object) — https://github.com/emosca-cnr/margheRita

## Examples

```
collapsed <- margheRita::collapse_tech_rep(mRList_urine)
```

## Evaluation signals

- The collapsed mRList contains exactly one sample per unique concatenation of 'class' and 'biological_rep' from the original annotation
- Downstream univariate() with test_method = 'anova' runs on the collapsed object and writes an anova.txt containing a test statistic and q-value for every feature
- The sample annotation file used as input carries all required columns (id, injection_order, batch, class, biological_rep, technical_rep); missing columns mean the collapse grouping is undefined
- Feature count (e.g. 604 features in the Urine dataset after filtering) is preserved through collapsing; only samples are merged, features are not removed

## Limitations

- The grouping is fixed to the concatenation of 'class' and 'biological_rep'; samples differing only in technical_rep are merged, so any other factor (e.g. batch) is not part of the collapse key and must be handled separately
- The article's Urine demonstration still required subsequent filtering steps (m/z decimal-value filtering, CV ratio samples vs QCs, imputation of NA values between 10%–25% of the feature minimum) before statistical testing; collapsing alone does not guarantee analysis-ready data

## Evidence

- [intro] the collapse of technical replicates takes place over all samples that share the same concatenation of "class" and "biological_rep" values: "the collapse of technical replicates (see below) takes place over all samples that share the same concatenation of "class" and "biological_rep" values."
- [intro] It requires two text files in tab-delimited format: "It requires two text files in tab-delimited format"
- [readme] simplified execution of parametric and non-parametric statistical tests over a large number of features: "simplified execution of parametric and non-parametric statistical tests over a large number of features;"
- [intro] The full version of the "Urine" dataset, which was used for margheRita assessment and to generate this documentation, is available at https://doi.org/10.5281/zenodo.11243781: "The full version of the "Urine" dataset, which was used for margheRita assessment and to generate this documentation, is available at https://doi.org/10.5281/zenodo.11243781"
