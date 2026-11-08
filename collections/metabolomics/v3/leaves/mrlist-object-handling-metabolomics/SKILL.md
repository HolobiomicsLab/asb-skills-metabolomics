---
name: mrlist-object-handling-metabolomics
description: Use when an mRList built from MS-Dial tab-delimited feature and sample-annotation
  files lacks a 'reference' column in mRList$metab_ann and probabilistic quotient
  normalization (PQN) is required on filtered LC-MS/MS metabolomic profiles.
when_to_use_negative:
- When mRList$metab_ann already contains a 'reference' column — calc_reference() is
  only needed to create the missing reference profile
- When the input is raw vendor data rather than a peak/feature table already processed
  by MS-Dial; margheRita expects two tab-delimited text files (feature data and sample
  annotation)
- When a normalization method other than PQN is required (margheRita also offers log
  and reference normalization)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: margheRita
  role: R package providing the mRList object, calc_reference() and normalize_profiles(method
    = 'pqn') used to build the missing reference profile and normalize the profiles
  repo: https://github.com/emosca-cnr/margheRita
- name: MS-Dial
  role: Generates the tab-delimited feature abundance file (features x samples) that
    is loaded into the mRList before normalization
- name: R
  role: Runtime environment in which the margheRita functions are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.06.20.599545
    title: 'MargheRita: streamlining MS-DIAL output analysis and metabolite identification
      in R'
  source_content_hash: sha256:d1695e014825c014e76381febc5d3b6f4cf7c88353c6a2008d6cb71275747078
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emosca-cnr/margheRita
  tools:
  - MS-Dial
  - R
  - margheRita
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# mrlist-object-handling-metabolomics

## Summary

Handles the margheRita mRList object — the container holding the feature abundance matrix and sample annotations — including computing a missing PQN reference profile with calc_reference() (average metabolite values and QC-sample medians) before calling normalize_profiles(method = 'pqn').

## When to use

Use when an mRList built from MS-Dial tab-delimited feature and sample-annotation files lacks a 'reference' column in mRList$metab_ann and probabilistic quotient normalization (PQN) is required on filtered LC-MS/MS metabolomic profiles.

## When NOT to use

- When mRList$metab_ann already contains a 'reference' column — calc_reference() is only needed to create the missing reference profile
- When the input is raw vendor data rather than a peak/feature table already processed by MS-Dial; margheRita expects two tab-delimited text files (feature data and sample annotation)
- When a normalization method other than PQN is required (margheRita also offers log and reference normalization)

## Inputs

- margheRita mRList object with feature abundance matrix and sample annotations derived from MS-Dial tab-delimited output
- Sample annotation file with columns id, injection_order, batch, class, biological_rep, technical_rep
- mRList$metab_ann metadata table (checked for a 'reference' column)

## Outputs

- mRList with normalized metabolite profiles after normalize_profiles(method = 'pqn')
- Normalized abundance matrix of features x samples, e.g. written to normalized_pqn.csv (tab- or comma-delimited)

## How to apply

Load the mRList object holding the feature abundance matrix and sample annotations, then inspect mRList$metab_ann for a 'reference' column. When the column is absent — the default situation for an MS-Dial-derived Urine-style dataset — generate it with calc_reference(), which builds the reference profile from average metabolite values and medians of QC samples. With the reference in place, run normalize_profiles(method = 'pqn') on the mRList; margheRita performs preprocessing steps such as m/z (mass defect) filtering, CV-ratio filtering between samples and QCs, NA filtering and imputation before normalization. Finally extract the normalized abundance matrix (features x samples) from the mRList and export it as a delimited table (e.g. normalized_pqn.csv).

## Related tools

- **margheRita** (R package providing the mRList object, calc_reference() and normalize_profiles(method = 'pqn') used to build the missing reference profile and normalize the profiles) — https://github.com/emosca-cnr/margheRita
- **MS-Dial** (Generates the tab-delimited feature abundance file (features x samples) that is loaded into the mRList before normalization)
- **R** (Runtime environment in which the margheRita functions are executed)

## Examples

```
mRList <- calc_reference(mRList); mRList <- normalize_profiles(mRList, method = 'pqn'); write.table(extract(mRList), 'normalized_pqn.csv', sep = ',', row.names = TRUE)
```

## Evaluation signals

- mRList$metab_ann contains a populated 'reference' column after calc_reference() runs
- The reference profile equals the medians of QC samples combined with average metabolite values as implemented in calc_reference()
- The exported matrix has one row per feature and one column per sample matching the input mRList dimensions (e.g. 604 features / 243 samples in the Urine dataset after filtering)
- Output file normalized_pqn.csv parses as a valid delimited features x samples table with the same feature identifiers as the input
- No NA values remain if imputation was run (NA values would have been replaced with a random number between 10%-25% of the feature minimum)

## Limitations

- calc_reference() is required only because the 'reference' column is missing; without it normalize_profiles(method = 'pqn') cannot proceed
- QC samples must be present in the sample annotations for the median-based reference computation
- PQN is described for LC-MS/MS untargeted metabolomic profiles; margheRita was assessed on a 'Urine' dataset (243 samples, 604 features) and DIA data
- The margheRita documentation notes 'No NAs: imputation not performed.' when no NA values are present, so imputation-related assumptions must be checked per dataset

## Evidence

- [intro] filtering by mass defects, filtering by coefficient of variation (samples vs QCs) and probabilistic quotient normalization;: "filtering by mass defects, filtering by coefficient of variation (samples vs QCs) and probabilistic quotient normalization;"
- [intro] It requires two text files in tab-delimited format: "It requires two text files in tab-delimited format"
- [intro] the imputation is performed replacing NA values with a random number, calculated between 10%-25% of the minimum value of the feature.: "the imputation is performed replacing NA values with a random number, calculated between 10%-25% of the minimum value of the feature."
- [intro] The R package margheRita addresses the complete workflow for metabolomic profiling in untargeted studies based on liquid chromatography (LC) coupled with tandem mass spectrometry (MS/MS).: "The R package margheRita addresses the complete workflow for metabolomic profiling in untargeted studies based on liquid chromatography (LC) coupled with tandem mass spectrometry (MS/MS)."
- [readme] a series of pre-processing functions (quality control, filtering and normalization) with a particular focus on methods specifically recommended for metabolomic profiles: "a series of pre-processing functions (quality control, filtering and normalization) with a particular focus on methods specifically recommended for metabolomic profiles"
