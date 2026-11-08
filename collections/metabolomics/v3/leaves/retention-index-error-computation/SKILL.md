---
name: retention-index-error-computation
description: Use when annotating features from an mRList object against a margheRita
  spectral library (e.g. RPShort positive mode) and you need per-match ppm, retention-time,
  and retention-index errors to support level-1 identifications.
when_to_use_negative:
- Features lack MS/MS information or the reference standard is absent from the selected
  library — level-1 confidence cannot be achieved, only lower-level annotation
- Data were acquired on a chromatographic column or polarity not covered by the margheRita
  libraries (HILIC, LipC8, pZIC, RPLong, RPShort in positive/negative modes)
- Input is raw instrument data rather than an mRList processed by upstream margheRita
  pre-processing (quality control, filtering, normalization)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: margheRita
  role: Provides metabolite_identification(), the in-house spectral libraries of reference
    standards, and the writer that exports the association table to metabolite_identification.xlsx
  repo: https://github.com/emosca-cnr/margheRita
- name: MS-Dial
  role: Generates the feature table upstream; its much larger library (~1e5 metabolites)
    is limited to level-2 identifications, in contrast to the margheRita in-house
    library used here
- name: R
  role: Execution environment for the margheRita metabolite_identification() workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.06.20.599545
    title: 'MargheRita: streamlining MS-DIAL output analysis and metabolite identification
      in R'
  source_content_hash: sha256:35359ce026dd427863c0bd0596425d5278ab823e89c10e5923e00fcc36a724be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emosca-cnr/margheRita
  tools:
  - MS-Dial
  - R
  - margheRita
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# retention-index-error-computation

## Summary

Computation of metabolite annotation confidence errors during margheRita metabolite_identification(), where each feature-metabolite match is reported with a mass-accuracy error in ppm, a retention-time error, and a retention-index error against the in-house spectral library of reference standards.

## When to use

Use when annotating features from an mRList object against a margheRita spectral library (e.g. RPShort positive mode) and you need per-match ppm, retention-time, and retention-index errors to support level-1 identifications.

## When NOT to use

- Features lack MS/MS information or the reference standard is absent from the selected library — level-1 confidence cannot be achieved, only lower-level annotation
- Data were acquired on a chromatographic column or polarity not covered by the margheRita libraries (HILIC, LipC8, pZIC, RPLong, RPShort in positive/negative modes)
- Input is raw instrument data rather than an mRList processed by upstream margheRita pre-processing (quality control, filtering, normalization)

## Inputs

- mRList object (normalized/filtered margheRita pre-processing output) with feature m/z, retention times, and MS/MS spectra
- margheRita reference spectral library entry for the chromatographic column and polarity (e.g. RPShort positive mode; columns covered: HILIC, LipC8, pZIC, RPLong, RPShort)

## Outputs

- Feature-metabolite association table with identification level, mass-accuracy error (ppm), retention-time error, and retention-index error per matched feature
- Excel file metabolite_identification.xlsx with the associations written to the 'associations' sheet

## How to apply

Load the normalized/filtered mRList produced by upstream margheRita pre-processing and the margheRita reference spectral library entry matching the chromatographic column and ion mode (e.g. RPShort in positive mode). Call metabolite_identification() so feature m/z, retention time, and MS/MS spectra are matched against the spectral library of reference standards; this is what enables identification up to level-1 confidence when MS/MS information is available in the library. Extract the resulting feature-metabolite association table, which carries the assigned identification level together with the mass-accuracy error (ppm), retention-time error, and retention-index error columns per matched feature. Write the table to the 'associations' sheet of metabolite_identification.xlsx (via the openxlsx/writexl writer used by margheRita). Judge correctness by checking that every reported match carries all three error columns and a level, and that errors are consistent with the matched reference standard's m/z, RT, and retention index.

## Related tools

- **margheRita** (Provides metabolite_identification(), the in-house spectral libraries of reference standards, and the writer that exports the association table to metabolite_identification.xlsx) — https://github.com/emosca-cnr/margheRita
- **MS-Dial** (Generates the feature table upstream; its much larger library (~1e5 metabolites) is limited to level-2 identifications, in contrast to the margheRita in-house library used here)
- **R** (Execution environment for the margheRita metabolite_identification() workflow)

## Examples

```
mRList <- metabolite_identification(mRList, spectral_library = RPShort_positive)
```

## Evaluation signals

- Association table contains, for each matched feature, an identification level plus mass-accuracy error (ppm), retention-time error, and retention-index error columns
- Features matched to library standards with available MS/MS information receive level-1 confidence; matches without MS/MS support do not exceed that level
- metabolite_identification.xlsx exists on disk with the association table under the 'associations' sheet
- Reported matches are restricted to the library corresponding to the chromatographic column/ion mode used (e.g. RPShort positive), not a mixed-library union
- Error magnitudes are computed per matched reference standard (ppm-scale mass errors), not left as NA for annotated features

## Limitations

- Level-1 identification is only possible when the required MS/MS information is available in the selected reference spectral library
- The in-house libraries are limited to ~800 metabolites across five column types; the larger MS-Dial library (~1e5) yields only level-2 identifications
- The workflow is especially advantageous for data-independent acquisition (DIA) with all MS/MS spectra at high resolution; other acquisition modes may be less well supported

## Evidence

- [other] Metabolite identification with margheRita is possible up to level-1 confidence, provided that the required MS/MS information is available in the selected reference spectral library.: [evidence span withheld — anchor status: not_found]
- [intro] level-1 identifications across five columns: "these libraries provideup to level 1 identifications in positive and negative modalities for "HILIC", "LipC8", "pZIC", "RPLong" and "RPShort" chromatographic columns"
- [readme] library rationale from README: "the R package margheRita enhances fragment matching accuracy by providing an original metabolite spectral library acquired in both polarities using different chromatographic columns"
- [other] association table content from workflow card: [evidence span withheld — anchor status: not_found]
- [other] output export from workflow card: [evidence span withheld — anchor status: not_found]
