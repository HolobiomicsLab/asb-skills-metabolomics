---
name: metabolite-annotation-level-assignment
description: Use when a filtered/normalized margheRita mRList object derived from
  MS-Dial output needs feature-metabolite annotation, and MS/MS spectra for features
  are available for matching against the RPShort positive-mode spectral library.
when_to_use_negative:
- When no MS/MS information for the features is present in the selected reference
  spectral library — margheRita identification is only possible up to level-1 provided
  the required MS/MS information is available in the library.
- When the goal is level-2 annotation for metabolites outside margheRita's ~800-metabolite
  in-house library and the broader MS-Dial library (~1e5 metabolites) is acceptable.
- When the mRList has not yet been pre-processed (filtered, imputed, normalized) —
  identification expects the normalized/filtered object from upstream margheRita steps.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: margheRita
  role: Provides metabolite_identification() and the in-house spectral libraries of
    reference standards for matching feature m/z, retention time, and MS/MS spectra
  repo: https://github.com/emosca-cnr/margheRita
- name: MS-Dial
  role: Generates the tab-delimited feature table consumed upstream; its MS/MS library
    (~1e5 metabolites) is the freely available alternative limited to level-2 identifications
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.06.20.599545
    title: 'MargheRita: streamlining MS-DIAL output analysis and metabolite identification
      in R'
  source_content_hash: sha256:da5df1e26376d689f2861353514a68d990b507cd362b0451f6ad4a1649459c5c
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# metabolite-annotation-level-assignment

## Summary

Assigns metabolite identifications to LC-MS/MS features by matching m/z, retention time, and MS/MS spectra against margheRita's in-house reference spectral library of ~800 standards, reaching up to Metabolomics Standards Initiative level-1 confidence (level 2 with the free MS-Dial library).

## When to use

Use when a filtered/normalized margheRita mRList object derived from MS-Dial output needs feature-metabolite annotation, and MS/MS spectra for features are available for matching against the RPShort positive-mode spectral library.

## When NOT to use

- When no MS/MS information for the features is present in the selected reference spectral library — margheRita identification is only possible up to level-1 provided the required MS/MS information is available in the library.
- When the goal is level-2 annotation for metabolites outside margheRita's ~800-metabolite in-house library and the broader MS-Dial library (~1e5 metabolites) is acceptable.
- When the mRList has not yet been pre-processed (filtered, imputed, normalized) — identification expects the normalized/filtered object from upstream margheRita steps.

## Inputs

- mRList object: normalized/filtered margheRita data structure from upstream pre-processing of MS-Dial output
- margheRita reference spectral library for the relevant column and mode (e.g. RPShort positive; libraries exist for HILIC, LipC8, pZIC, RPLong, RPShort in both polarities)
- Tab-delimited feature data file from MS-Dial containing feature identifiers, m/z, retention times, MS/MS spectra, and abundances (with a sample annotation file)

## Outputs

- Feature-metabolite association table with assigned identification level (up to level 1), mass-accuracy error in ppm, retention-time error, and retention-index error per matched feature
- Excel file metabolite_identification.xlsx with results on the 'associations' sheet

## How to apply

Load the normalized/filtered mRList produced by upstream margheRita pre-processing (after QC, mass-defect filtering, CV ratio filtering, imputation, and probabilistic quotient normalization). Select the margheRita reference spectral library matching the chromatographic column and ionization mode of the experiment — e.g. RPShort in positive mode — choosing between margheRita's in-house library (~800 metabolites, up to level 1) and the larger MS-Dial library (~1e5 metabolites, limited to level 2). Call metabolite_identification() on the mRList so that feature m/z, retention time, and MS/MS spectra are matched against the library of reference standards. Extract the feature-metabolite association table, which carries the assigned identification level together with mass-accuracy error (ppm), retention-time error, and retention-index error columns per matched feature. Write the table to the 'associations' sheet of metabolite_identification.xlsx (e.g. via the openxlsx/writexl writer used by margheRita).

## Related tools

- **margheRita** (Provides metabolite_identification() and the in-house spectral libraries of reference standards for matching feature m/z, retention time, and MS/MS spectra) — https://github.com/emosca-cnr/margheRita
- **MS-Dial** (Generates the tab-delimited feature table consumed upstream; its MS/MS library (~1e5 metabolites) is the freely available alternative limited to level-2 identifications)

## Evaluation signals

- The association table includes an assigned identification level column, with level-1 assignments possible only when MS/MS information was present in the selected library.
- Mass-accuracy error (ppm), retention-time error, and retention-index error columns are populated for each matched feature, enabling accuracy sanity checks.
- Matching is performed against the library entry consistent with the experiment's column and mode (e.g. RPShort positive) — mismatched column/mode libraries should yield no level-1 assignments.
- Results are persisted as the 'associations' sheet in metabolite_identification.xlsx.
- Identifications are restricted to library content: margheRita's in-house library (~800 metabolites) vs MS-Dial's (~1e5) explains differences in the number of annotated features.

## Limitations

- Level-1 identification requires the required MS/MS information to be present in the selected reference spectral library; without it, identifications cannot reach level-1 confidence.
- The margheRita in-house spectral library covers ~800 metabolites; larger coverage via the MS-Dial library (~1e5) is limited to level 2 identifications.
- Libraries are column- and mode-specific (HILIC, LipC8, pZIC, RPLong, RPShort; positive and negative modalities), so the correct library must be selected for the chromatographic setup.

## Evidence

- [intro] in-house library level-1 scope: "metabolite annotation up to level-1, based on in-house spectral libraries as well as freely available libraries;"
- [intro] column- and mode-specific libraries: "these libraries provideup to level 1 identifications in positive and negative modalities for "HILIC", "LipC8", "pZIC", "RPLong" and "RPShort" chromatographic columns"
- [intro] MS-Dial library limitation: "**MS-Dial**, which covers a much larger set of metabolites ($10^5$), but is limited to level 2 identifications in positive and negative modalities."
- [readme] README library coverage: "spectral libraries that covers 4 different chromatographic column types: RP-C18, HILIC, RP-C8 and pZIC-HILIC Zwitterionic."
