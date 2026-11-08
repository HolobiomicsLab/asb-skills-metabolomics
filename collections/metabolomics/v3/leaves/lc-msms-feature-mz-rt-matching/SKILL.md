---
name: lc-msms-feature-mz-rt-matching
description: Use when a preprocessed margheRita mRList (MS-Dial-exported features
  with m/z, retention time and MS/MS spectra) must be matched against a reference
  spectral library to produce feature-metabolite associations up to level-1 confidence.
when_to_use_negative:
- The data has not been pre-processed through margheRita's QC/filtering/normalization
  pipeline — there is no mRList with feature m/z, RT and MS/MS to match
- Broad coverage matters more than identification level — the MS-Dial library (~10^5
  metabolites) is the larger choice, but it is limited to level-2 identifications
- The chromatographic column / ion-mode combination is not covered by any of the five
  margheRita libraries (HILIC, LipC8, pZIC, RPLong, RPShort) in the needed polarity
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: margheRita
  role: Provides the metabolite_identification() function and the in-house reference
    spectral libraries of standards that the feature m/z/RT/MS-MS matching runs against
  repo: github.com/emosca-cnr/margheRita
- name: MS-Dial
  role: Upstream source of the feature table and MS/MS spectra exported for matching,
    and of a larger but level-2-limited MS/MS library
  repo: http://prime.psc.riken.jp/Metabolomics_Software/MS-DIAL/
- name: R
  role: Runtime environment in which the margheRita identification call is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.06.20.599545
    title: 'MargheRita: streamlining MS-DIAL output analysis and metabolite identification
      in R'
  source_content_hash: sha256:3630e76fee52de79b85826c2f51a68041c678f0519965714ec97cd742cee8492
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS-Dial
  - R
  - margheRita
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lc-msms-feature-mz-rt-matching

## Summary

Assign metabolite identities to LC-MS/MS features by matching feature m/z, retention time and MS/MS spectra against a reference spectral library of standards, yielding level-1 confident identifications in margheRita.

## When to use

Use when a preprocessed margheRita mRList (MS-Dial-exported features with m/z, retention time and MS/MS spectra) must be matched against a reference spectral library to produce feature-metabolite associations up to level-1 confidence.

## When NOT to use

- The data has not been pre-processed through margheRita's QC/filtering/normalization pipeline — there is no mRList with feature m/z, RT and MS/MS to match
- Broad coverage matters more than identification level — the MS-Dial library (~10^5 metabolites) is the larger choice, but it is limited to level-2 identifications
- The chromatographic column / ion-mode combination is not covered by any of the five margheRita libraries (HILIC, LipC8, pZIC, RPLong, RPShort) in the needed polarity

## Inputs

- Normalized/filtered mRList object from margheRita pre-processing (feature identifiers, m/z, retention times, MS/MS spectra, abundances)
- margheRita reference spectral library for the appropriate column type and ion mode (e.g. RPShort, positive mode; ~800 metabolites across HILIC, LipC8, pZIC, RPLong, RPShort)
- Optionally, freely available libraries such as the MS-Dial MS/MS library (~10^5 metabolites, level-2 identifications)

## Outputs

- Feature-metabolite association table with assigned identification level (up to level-1), mass-accuracy error (ppm), retention-time error and retention-index error per matched feature
- Excel file metabolite_identification.xlsx with the association table on the 'associations' sheet

## How to apply

Take the normalized/filtered mRList produced by upstream margheRita pre-processing (QC, mass-defect and CV filtering, probabilistic quotient normalization) and load the margheRita reference spectral library entry matching the experiment's chromatographic column (HILIC, LipC8, pZIC, RPLong or RPShort) and ion mode (positive/negative). Call the margheRita function metabolite_identification() on the mRList with the selected library; the function matches feature m/z, retention time and MS/MS spectra against the spectral library of reference standards. Level-1 confidence is achievable only where the required MS/MS information is present in the selected library — the MS-Dial library covers ~10^5 metabolites but only supports level-2. Extract the resulting association table, which carries the identification level together with mass-accuracy error (ppm), retention-time error and retention-index error columns for each matched feature. Write the table to the 'associations' sheet of the Excel file metabolite_identification.xlsx (e.g. via the openxlsx/writexl writer used by margheRita).

## Related tools

- **margheRita** (Provides the metabolite_identification() function and the in-house reference spectral libraries of standards that the feature m/z/RT/MS-MS matching runs against) — github.com/emosca-cnr/margheRita
- **MS-Dial** (Upstream source of the feature table and MS/MS spectra exported for matching, and of a larger but level-2-limited MS/MS library) — http://prime.psc.riken.jp/Metabolomics_Software/MS-DIAL/
- **R** (Runtime environment in which the margheRita identification call is executed)

## Examples

```
mRList <- metabolite_identification(mRList, lib_RPShort_pos)
```

## Evaluation signals

- The association table contains matched features carrying identification level, mass-accuracy error (ppm), retention-time error and retention-index error columns
- Assigned identifications do not exceed level-1 confidence, and level-1 assignments are only made where the library contains the required MS/MS spectra
- The column/ion-mode library used matches the acquisition (one of HILIC, LipC8, pZIC, RPLong, RPShort in the correct polarity)
- metabolite_identification.xlsx exists on disk with a populated 'associations' sheet
- Matched-feature counts are consistent with the ~800-metabolite margheRita library coverage rather than the ~10^5 MS-Dial library

## Limitations

- Level-1 identifications are only possible when the required MS/MS information is present in the selected reference spectral library
- The MS-Dial library, although much larger (~10^5 metabolites), is limited to level-2 identifications
- The margheRita libraries cover only five chromatographic column types (HILIC, LipC8, pZIC, RPLong, RPShort) in positive and negative polarities

## Evidence

- [intro] metabolite annotation up to level-1, based on in-house spectral libraries as well as freely available libraries;: "metabolite annotation up to level-1, based on in-house spectral libraries as well as freely available libraries;"
- [intro] fragment matching accuracy via original library: "margheRita enhances fragment matching accuracy by providing an original metabolite spectral library acquired in both polarities using different chromatographic columns"
- [readme] library column coverage: "spectral libraries that covers 4 different chromatographic column types: RP-C18, HILIC, RP-C8 and pZIC-HILIC Zwitterionic."
- [readme] pre-processing prerequisite: "a series of pre-processing functions (quality control, filtering and normalization) with a particular focus on methods specifically recommended for metabolomic profiles"
