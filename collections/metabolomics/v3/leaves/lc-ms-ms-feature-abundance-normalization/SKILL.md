---
name: lc-ms-ms-feature-abundance-normalization
description: Use when a margheRita mRList from MS-Dial LC-MS/MS profiling (e.g. the
  Urine dataset) has passed QC/filtering and sample abundances must be normalized,
  with PQN or log/reference methods.
when_to_use_negative:
- mRList$metab_ann already contains a 'reference' column, so calc_reference() should
  not regenerate the PQN reference profile
- The dataset lacks QC samples, since the default reference profile is built from
  medians of QC samples
- The data is a raw MS-Dial export that has not yet been loaded and quality-controlled
  (e.g. m/z, NA and CV filtering) in margheRita
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: margheRita
  role: Provides calc_reference() and normalize_profiles(method = 'pqn') to build
    the PQN reference profile (average metabolite values, QC medians) and normalize
    the mRList abundance matrix
  repo: https://github.com/emosca-cnr/margheRita
- name: R
  role: Runtime environment in which the margheRita normalization functions are executed
    and the normalized matrix is exported
- name: MS-Dial
  role: Upstream tool that generates the tab-delimited feature abundance files loaded
    into margheRita before normalization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.06.20.599545
    title: 'MargheRita: streamlining MS-DIAL output analysis and metabolite identification
      in R'
  source_content_hash: sha256:61e0ef8bc0effa935b1dee1518ba00673da765ecc495461be23f68c268b8a057
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emosca-cnr/margheRita
  tools:
  - MS-Dial
  - R
  - margheRita
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# lc-ms-ms-feature-abundance-normalization

## Summary

Normalization of LC-MS/MS untargeted metabolomics feature abundance matrices using the margheRita R package, in particular probabilistic quotient normalization (PQN) of a filtered mRList object derived from MS-Dial output.

## When to use

Use when a margheRita mRList from MS-Dial LC-MS/MS profiling (e.g. the Urine dataset) has passed QC/filtering and sample abundances must be normalized, with PQN or log/reference methods.

## When NOT to use

- mRList$metab_ann already contains a 'reference' column, so calc_reference() should not regenerate the PQN reference profile
- The dataset lacks QC samples, since the default reference profile is built from medians of QC samples
- The data is a raw MS-Dial export that has not yet been loaded and quality-controlled (e.g. m/z, NA and CV filtering) in margheRita

## Inputs

- margheRita mRList object with feature abundance matrix (features x samples) and sample annotations (metab_ann), derived from MS-Dial tab-delimited output
- QC sample annotations in the sample table (class column identifying QC samples), used to build the PQN reference profile

## Outputs

- Updated mRList with PQN-normalized feature abundance values
- Normalized abundance matrix exported as features x samples delimited CSV (e.g. normalized_pqn.csv)

## How to apply

Load the mRList object holding the feature abundance matrix and sample annotations. For probabilistic quotient normalization, inspect mRList$metab_ann for a 'reference' column: when absent, run calc_reference(), which builds the reference profile from average metabolite values and medians of the QC samples. Then call normalize_profiles(method = 'pqn') on the mRList; margheRita also supports log and reference-based normalization within the same pre-processing block alongside mass-defect filtering and CV (samples vs QCs) filtering. Finally extract the normalized abundance matrix (features x samples) from the updated mRList and export it as a delimited table such as normalized_pqn.csv.

## Related tools

- **margheRita** (Provides calc_reference() and normalize_profiles(method = 'pqn') to build the PQN reference profile (average metabolite values, QC medians) and normalize the mRList abundance matrix) — https://github.com/emosca-cnr/margheRita
- **R** (Runtime environment in which the margheRita normalization functions are executed and the normalized matrix is exported)
- **MS-Dial** (Upstream tool that generates the tab-delimited feature abundance files loaded into margheRita before normalization)

## Examples

```
mRList <- calc_reference(mRList); mRList <- normalize_profiles(mRList, method = "pqn"); write.csv(mRList$metab, "normalized_pqn.csv")
```

## Evaluation signals

- mRList$metab_ann contains a 'reference' column after calc_reference() when it was missing beforehand
- The exported normalized_pqn.csv has the same features x samples dimensions as the filtered abundance matrix, with no NA entries after imputation
- PQN quotients are computed against a reference profile whose values equal the averages of metabolite abundances and medians of QC samples
- Sample abundance distributions after PQN are comparable in scale across samples (e.g. via PCA or median profile inspection), consistent with quotient normalization

## Limitations

- Without a 'reference' column in mRList$metab_ann, PQN depends on calc_reference() being run first; otherwise the call fails
- The default reference profile relies on QC samples being present and well-measured; imputation (NA replaced by a random number between 10%-25% of the feature minimum) can influence the values entering normalization
- The margheRita package API remains experimental and breaking changes are possible, as noted for closely related R metabolomics tooling

## Evidence

- [intro] such as filtering by mass defects, filtering by coefficient of variation (samples vs QCs) and probabilistic quotient normalization;: "such as filtering by mass defects, filtering by coefficient of variation (samples vs QCs) and probabilistic quotient normalization;"
- [readme] a series of pre-processing functions (quality control, filtering and normalization): "a series of pre-processing functions (quality control, filtering and normalization)"
- [other] When the required 'reference' column is missing from mRList$metab_ann, the function calc_reference() creates it using average metabolite values and medians of QC samples: [evidence span withheld — anchor status: not_found]
