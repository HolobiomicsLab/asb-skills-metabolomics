---
name: probabilistic-quotient-normalization
description: Use when an mRList of MS-Dial-derived Urine or similar LC-MS/MS feature
  abundances has been filtered (m/z, NA, CV vs QC) and needs PQN, but mRList$metab_ann
  lacks the 'reference' column.
when_to_use_negative:
- Profiles that have already been normalized (e.g. by log or reference-sample normalization)
  would be distorted by a second PQN pass
- Data has not been filtered/imputed first (m_z_filtering, filter_NA, CV vs QC, imputation)
  — PQN quotients would be computed on low-quality or NA-containing features
- The task is peak picking or drift correction in a different pipeline (e.g. notame's
  cubic spline drift correction) rather than margheRita-based normalization
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: margheRita
  role: provides calc_reference() to build the PQN reference profile (average metabolite
    values, QC medians) and normalize_profiles(method = 'pqn') to apply probabilistic
    quotient normalization on the mRList
  repo: https://github.com/emosca-cnr/margheRita
- name: R
  role: runtime environment in which the margheRita mRList workflow is executed and
    the normalized matrix is written to CSV
- name: MS-Dial
  role: upstream tool that generates the feature abundance table consumed by margheRita
    for PQN normalization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.06.20.599545
    title: 'MargheRita: streamlining MS-DIAL output analysis and metabolite identification
      in R'
  source_content_hash: sha256:0eb17f1d0262e39a499fdd7df8db696bd23fa78b5cf69c5feb5b2ec6078a6c03
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# probabilistic-quotient-normalization

## Summary

Apply probabilistic quotient normalization (PQN) to filtered LC-MS/MS metabolomic profiles with the margheRita function normalize_profiles(method = 'pqn'), generating the required reference profile via calc_reference() when no 'reference' column exists in mRList$metab_ann.

## When to use

Use when an mRList of MS-Dial-derived Urine or similar LC-MS/MS feature abundances has been filtered (m/z, NA, CV vs QC) and needs PQN, but mRList$metab_ann lacks the 'reference' column.

## When NOT to use

- Profiles that have already been normalized (e.g. by log or reference-sample normalization) would be distorted by a second PQN pass
- Data has not been filtered/imputed first (m_z_filtering, filter_NA, CV vs QC, imputation) — PQN quotients would be computed on low-quality or NA-containing features
- The task is peak picking or drift correction in a different pipeline (e.g. notame's cubic spline drift correction) rather than margheRita-based normalization

## Inputs

- margheRita mRList object with feature abundance matrix and metab_ann sample annotations
- MS-Dial-generated tab-delimited feature data file (feature identifiers, m/z, retention times, MS/MS spectra, abundances)
- tab-delimited sample annotation file with columns id, injection_order, batch, class, biological_rep, technical_rep

## Outputs

- mRList with PQN-normalized abundance profiles
- features x samples normalized matrix exported as normalized_pqn.csv (tab- or comma-delimited)

## How to apply

Load the margheRita mRList object holding the feature abundance matrix and sample annotations (two tab-delimited inputs: feature data with m/z, retention times, MS/MS spectra, abundances; and a sample annotation file with id, injection_order, batch, class, biological_rep, technical_rep). After pre-processing (m_z_filtering, filter_NA, imputation replacing NAs with 10%-25% of the feature minimum, CV-ratio filtering vs QCs with default threshold 1), inspect metab_ann for a 'reference' column; if absent, call calc_reference(), which builds the reference from average metabolite values and medians of QC samples. Then call normalize_profiles(method = 'pqn') to scale each sample's features by sample-specific quotients against that reference. Extract the normalized features x samples matrix from the mRList and export it as a tab- or comma-delimited table (e.g. normalized_pqn.csv).

## Related tools

- **margheRita** (provides calc_reference() to build the PQN reference profile (average metabolite values, QC medians) and normalize_profiles(method = 'pqn') to apply probabilistic quotient normalization on the mRList) — https://github.com/emosca-cnr/margheRita
- **R** (runtime environment in which the margheRita mRList workflow is executed and the normalized matrix is written to CSV)
- **MS-Dial** (upstream tool that generates the feature abundance table consumed by margheRita for PQN normalization)

## Examples

```
mRList <- calc_reference(mRList); mRList <- normalize_profiles(mRList, method = 'pqn'); write.csv(mRList$metab, 'normalized_pqn.csv')
```

## Evaluation signals

- A 'reference' column exists in mRList$metab_ann after calc_reference() (or was already present), so normalize_profiles(method = 'pqn') runs without a missing-reference error
- The exported normalized_pqn.csv is a features x samples table whose sample counts match the filtered mRList (e.g. 243/243 samples, 604/604 features retained in the Urine example)
- Normalized abundances differ from the filtered input only by per-sample scaling quotients; feature-wise medians/means of quotients across features are near 1 per sample
- No NAs remain (imputation replaced them with 10%-25% of each feature's minimum) so PQN quotients are computable for every feature
- Re-running calc_reference() + normalize_profiles(method = 'pqn') on the same mRList reproduces an identical normalized matrix

## Limitations

- PQN in margheRita depends on a reference profile: without a 'reference' column in mRList$metab_ann, calc_reference() must derive one from average metabolite values and QC medians, so normalization quality follows QC sample representativeness
- PQN is applied after specific margheRita pre-processing (mass-defect filtering, CV ratio samples-vs-QC threshold default 1, NA filtering, imputation); skipping these changes the quotients
- The package targets untargeted LC-MS/MS profiling (notably DIA/SWATH data); its PQN implementation assumes margheRita's mRList object structure and MS-Dial-style inputs

## Evidence

- [other] PQN reference construction when column is missing: [evidence span withheld — anchor status: not_found]
- [intro] PQN as a recommended metabolomic pre-processing method: "such as filtering by mass defects, filtering by coefficient of variation (samples vs QCs) and probabilistic quotient normalization;"
- [readme] pre-processing focus of the package: "a series of pre-processing functions (quality control, filtering and normalization) with a particular focus on methods specifically recommended for metabolomic profiles"
- [intro] workflow context: LC-MS/MS untargeted profiling: "The R package margheRita addresses the complete workflow for metabolomic profiling in untargeted studies based on liquid chromatography (LC) coupled with tandem mass spectrometry (MS/MS)."
