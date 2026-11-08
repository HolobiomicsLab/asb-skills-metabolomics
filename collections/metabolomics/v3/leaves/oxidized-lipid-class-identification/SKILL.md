---
name: oxidized-lipid-class-identification
description: Use when annotating ddMS2-topN or AIF UHPLC-HRMS/MS features that may
  derive from oxidized fatty acids or other oxidized lipids absent from standard libraries
  like LipidBlast (26 classes) — e.g. features matching precursor m/z of O/OH/OOH
  or aldehyde/carboxylic-acid FA species.
when_to_use_negative:
- Input data are Waters vendor files — LipidMatch does not currently support Waters
  files
- The lipid type of interest is not in the shipped libraries and the user has not
  built a custom fragmentation library in Excel
- Full structural resolution beyond class/fatty-acyl rule-based annotation is required
  (e.g. regioisomer or oxidized-site localization the fragment rules do not confirm)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipidMatch
  role: R-based rule-based engine that matches experimental fragment m/z to in silico
    oxidized-lipid library fragments and appends identifications to feature tables
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MSConvert
  role: Converts vendor MS/MS peak-picked data to the .ms2 file format LipidMatch
    consumes
- name: MS-DIAL
  role: Alternative/adjunct peak-picking and identification software used for feature
    tables and cross-software corroboration of annotations
- name: R
  role: Language in which LipidMatch is written (gWidgets/tcltk GUI) and used to generate
    fatty-acid combination lists and tally library contents
  repo: https://github.com/GarrettLab-UF/LipidMatch
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:4e15d4bcbb1c0a45128091fa6564a58a63748ef8df3be68162696f92e0a36231
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - LipidMatch
  - MS-DIAL
  - MSConvert
  - R
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# oxidized-lipid-class-identification

## Summary

Identifies oxidized lipid classes (O/OH/OOH additions and short-chain CHO/COOH cleavage products) in untargeted LC-MS/MS data by matching experimental fragments against LipidMatch's unique in silico oxidized-lipid fragmentation libraries using rule-based, user-selectable required fragments.

## When to use

Use when annotating ddMS2-topN or AIF UHPLC-HRMS/MS features that may derive from oxidized fatty acids or other oxidized lipids absent from standard libraries like LipidBlast (26 classes) — e.g. features matching precursor m/z of O/OH/OOH or aldehyde/carboxylic-acid FA species.

## When NOT to use

- Input data are Waters vendor files — LipidMatch does not currently support Waters files
- The lipid type of interest is not in the shipped libraries and the user has not built a custom fragmentation library in Excel
- Full structural resolution beyond class/fatty-acyl rule-based annotation is required (e.g. regioisomer or oxidized-site localization the fragment rules do not confirm)

## Inputs

- Feature table (m/z + retention time) per polarity from MZmine, XCMS, or MS-DIAL peak picking
- MS/MS spectra converted to .ms2 file format with MSConvert
- In silico oxidized-lipid fragmentation libraries in .csv format (e.g. from 2017_6_14_LipidMatch_Distribution.zip)
- User parameters: MS1 m/z tolerance (Da), RT and MS/MS m/z tolerances, fragment ppm tolerance, minimum fragment intensity and scan thresholds, required-fragment flags per lipid type

## Outputs

- Feature tables appended with oxidized lipid identifications
- Identification level (1-4) per annotation: class + fatty acyl, DIA, class-only, or precursor m/z only
- Ranked candidate lipid list per feature by summed fragment intensity
- Per-fragment 0/1 pass/fail matrix against intensity and scan thresholds

## How to apply

Generate or reuse in silico libraries in Excel/CSV built from lists of 39 endogenous fatty acids and 214 potential oxidized fatty acids (126 long-chain O/OH/OOH additions plus 88 short-chain CHO/COOH cleavage products drawn from LIPID MAPS). Match feature m/z values against in silico precursor ions using a user-defined Da tolerance at MS1, then find .ms2 scans within user-defined RT and m/z tolerances of each feature. Match experimental fragments to in silico fragment m/z within a ppm tolerance window, assign each fragment 0/1 based on user-defined minimum intensity and scan-count thresholds, and require the class-specific fragments the user marked as necessary for identification. Assign identification level 1-4 (class + fatty acyl, DIA, class-only, or precursor m/z only) and rank candidate oxidized lipid identifications by the summed intensity of all exact-mass-matched fragments.

## Related tools

- **LipidMatch** (R-based rule-based engine that matches experimental fragment m/z to in silico oxidized-lipid library fragments and appends identifications to feature tables) — https://github.com/GarrettLab-UF/LipidMatch
- **MSConvert** (Converts vendor MS/MS peak-picked data to the .ms2 file format LipidMatch consumes)
- **MS-DIAL** (Alternative/adjunct peak-picking and identification software used for feature tables and cross-software corroboration of annotations)
- **R** (Language in which LipidMatch is written (gWidgets/tcltk GUI) and used to generate fatty-acid combination lists and tally library contents) — https://github.com/GarrettLab-UF/LipidMatch

## Examples

```
libs <- lapply(list.files('2017_6_14_LipidMatch_Distribution', pattern='csv$', full.names=TRUE), read.csv); oxidized <- subset(do.call(rbind, libs), grepl('O-|OOH|OH', Class))
```

## Evaluation signals

- Every oxidized-lipid identification has the user-required class-specific fragments assigned 1 (above minimum intensity and scan-count thresholds)
- Matched fragment m/z values fall within the user-defined ppm tolerance of in silico fragment exact masses
- Identification levels are consistent with evidence: 1 only when class AND fatty acyl fragments are observed; 3 for class-only; 4 for precursor m/z only
- Oxidized-FA library composition checks out: 214 potential oxidized fatty acids = 126 long-chain O/OH/OOH additions + 88 short-chain CHO/COOH cleavage products, built from 39 endogenous fatty acids with redundant permutations (e.g. 18:0_20:0 vs 20:0_18:0) excluded
- Cross-software corroboration benchmark: ~92% of LipidMatch fatty-acyl identifications in positive mode and 98% in negative mode were corroborated by MS-DIAL or Greazy in the published Red Cross plasma comparison

## Limitations

- Oxidized-lipid rules depend on user-selected required fragments — omitting class-necessary fragments will allow false class assignments
- Coverage depends on the shipped/user-built libraries; users must develop their own fragmentation libraries in Excel for full customization
- Tested only on Windows 7 through 10 and pinned to R 3.3.3; depends on external MSConvert and peak-picking software
- Key artifacts (libraries, software zip) are provided only as Additional files (2017_6_14_LipidMatch_Distribution.zip) or via secim.ufl.edu, not archived in a public repository

## Evidence

- [abstract] Unique open-source oxidized-lipid libraries exist in LipidMatch: "Unique fragmentation libraries, compared to other open source software, include oxidized lipids, bile acids, sphingosines, and previously uncharacterized adducts, including ammoniated cardiolipins."
- [methods] Short-chain oxidized FA list generated from LIPID MAPS cleavage: "A list of 88 potential short chain oxidized fatty acids were generated by cleavage of unsaturated fatty acids contained in LIPID MAPS and addition of a terminal CHO (aldehyde) or COOH (carboxylic"
- [methods] Rule-based requirement of class-necessary fragments: "Lipids are identified if they contain the necessary observed fragments."
- [methods] MS1 exact mass matching with user-defined Da tolerance: "LipidMatch performs exact mass matching at the MS1 level between in silico precursor ions and each features m/z using a user defined m/z tolerance (Da)"
- [methods] Ranking by summed fragment intensity: "the lipids are ranked by the summed intensity of all their fragments with in silico fragment exact mass matches"
- [readme] README description of the matching mechanism: "LipidMatch identifications are obtained by matching experimental fragment m/z values with simulated library m/z values using in-silico fragmentation libraries"
