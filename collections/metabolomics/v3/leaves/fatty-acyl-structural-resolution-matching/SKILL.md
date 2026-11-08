---
name: fatty-acyl-structural-resolution-matching
description: Use when annotating UHPLC-HRMS/MS features (ddMS2-topN, AIF, or targeted
  MS/MS from Q-Exactive Orbitrap or Agilent/Bruker/SCIEX Q-TOF) to lipid species where
  identifications should be resolved to fatty acyl constituent level, not just lipid
  class or precursor mass.
when_to_use_negative:
- Input is raw vendor files that have not been feature-picked or converted to .ms2
  — LipidMatch requires feature tables and .ms2 MS/MS files, not raw data (Waters
  files are not supported)
- The goal is identification by precursor m/z alone or spectral-library search of
  experimental MS/MS against LipidBlast-type spectra — use a spectral library search
  instead; LipidMatch matches fragments against in silico libraries with rule-based
  confirmation
- Direct identification of lipids outside the library scope without building a custom
  fragmentation library in Excel first
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: LipidMatch
  role: Core R tool performing exact-mass precursor matching, fragment matching against
    in silico libraries, rule-based identification, ID-level assignment, and ranking;
    append IDs to feature tables
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MSConvert
  role: Peak picking of MS/MS data and conversion to .ms2 file format required as
    LipidMatch input
- name: MZmine
  role: Feature (peak) picking of full-scan data to produce the feature tables LipidMatch
    annotates; other options include XCMS, MS-DIAL, Compound Discoverer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:81bb7c9b4f380b409d99e968d78c4af17732f1850b6c2615dd5ea40149cee8bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - CombineSoftwareIDs.R
  - Greazy
  - LipidMatch
  - MS-DIAL
  - MSConvert
  - MZmine
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9583333333333334
n_claims: 24
---

# fatty-acyl-structural-resolution-matching

## Summary

Rule-based lipid identification at fatty acyl constituent structural resolution: match experimental MS/MS fragment m/z values against in silico fragmentation libraries in LipidMatch (R), requiring user-selected diagnostic fragments so that lipids are annotated by class and fatty acyl composition rather than precursor m/z alone.

## When to use

Use when annotating UHPLC-HRMS/MS features (ddMS2-topN, AIF, or targeted MS/MS from Q-Exactive Orbitrap or Agilent/Bruker/SCIEX Q-TOF) to lipid species where identifications should be resolved to fatty acyl constituent level, not just lipid class or precursor mass.

## When NOT to use

- Input is raw vendor files that have not been feature-picked or converted to .ms2 — LipidMatch requires feature tables and .ms2 MS/MS files, not raw data (Waters files are not supported)
- The goal is identification by precursor m/z alone or spectral-library search of experimental MS/MS against LipidBlast-type spectra — use a spectral library search instead; LipidMatch matches fragments against in silico libraries with rule-based confirmation
- Direct identification of lipids outside the library scope without building a custom fragmentation library in Excel first

## Inputs

- Feature tables per polarity from MZmine, XCMS, MS-DIAL, or Compound Discoverer
- MS/MS peak-picked spectra converted to .ms2 file format via MSConvert
- In silico lipid fragmentation libraries in .csv format (user-selectable or user-generated in Excel)
- User parameters: m/z tolerance (Da, MS1), RT and m/z tolerances for scan matching, fragment ppm tolerance, minimum fragment intensity and scan-count thresholds, required fragments per lipid type

## Outputs

- Feature tables with appended lipid identifications annotated by lipid class and fatty acyl constituents
- Identification level assignments (1–4) per feature
- Ranked candidate lipid identifications per feature by summed fragment intensity

## How to apply

Generate or load in silico fragmentation libraries (CSV) covering the lipid types of interest, then run LipidMatch: (1) exact-mass match in silico precursor ions to each feature's m/z with a user-defined m/z tolerance in Da; (2) find MS/MS scans from .ms2 files within user-defined retention-time and m/z tolerances of the feature; (3) match experimental fragments to in silico fragment m/z within a ppm tolerance window and count scans containing each fragment; (4) assign each fragment 1 if above user-defined minimum intensity and scan thresholds, else 0; (5) identify a lipid only if all user-selected necessary fragments for that lipid type are observed, then assign identification level 1 (class plus fatty acyl constituents), 2 (data-independent analysis), 3 (class only), or 4 (precursor m/z only); (6) rank multiple candidate lipids per feature by summed intensity of all exact-mass-matched fragments. Before running, features must be picked with MZmine/XCMS/MS-DIAL and MS/MS peak-picked and converted to .ms2 with MSConvert, organized into the directory structure expected by LipidMatch. The rationale is that rule-based, fragment-required matching gives correct annotation at appropriate structural resolution, including handling co-eluting isobaric molecules.

## Related tools

- **LipidMatch** (Core R tool performing exact-mass precursor matching, fragment matching against in silico libraries, rule-based identification, ID-level assignment, and ranking; append IDs to feature tables) — https://github.com/GarrettLab-UF/LipidMatch
- **MSConvert** (Peak picking of MS/MS data and conversion to .ms2 file format required as LipidMatch input)
- **MZmine** (Feature (peak) picking of full-scan data to produce the feature tables LipidMatch annotates; other options include XCMS, MS-DIAL, Compound Discoverer)

## Examples

```
Rscript LipidMatch.R  # enter m/z (Da) tolerance, RT/m/z scan-match tolerances, fragment ppm window, intensity/scan thresholds, and required fragments per lipid type in the gWidgets/tcltk dialogue boxes
```

## Evaluation signals

- Identifications carry an identification level 1–4, and level-1 IDs should specify both lipid class and fatty acyl constituents, not class only or precursor m/z only
- Every assigned lipid must contain all user-selected necessary fragments observed above the minimum intensity and scan-count thresholds (fragment assigned 1, not 0)
- When multiple lipids are candidates for one feature, ranking by summed fragment intensity should order them; ties or missing rank ordering indicate misconfiguration
- Cross-software corroboration check: for features identified by LipidMatch, MS-DIAL, and Greazy, fatty acyl constituent agreement with at least one other software should reach ~98% in negative and ~92% in positive ion mode, and LipidMatch should yield the most identifications per polarity (e.g., 159 negative / 1401 positive for Red Cross plasma)
- Fragment-to-feature association must respect the user-defined RT and m/z tolerances; IDs appearing from scans outside those windows indicate a tolerance misconfiguration

## Limitations

- Waters files are not currently supported
- Software tested only on Windows 7 through 10 and pinned to R version 3.3.3; depends on external peak-picking software (MZmine, XCMS, MS-DIAL or other) and MSConvert or equivalent for .ms2 generation
- Identification by class only (level 3) or precursor m/z only (level 4) occurs when fatty acyl-diagnostic fragments are absent; structural resolution depends on which fragments were observed
- Key artifacts (software zip, lipid libraries) were distributed only as additional files, not archived in a public repository at publication time
- Users must develop their own fragmentation libraries in Excel for full customization of required fragments

## Evidence

- [abstract] Rule-based identification allows for correct annotation of lipids based on the fragments observed: "Rule-based identification allows for correct annotation of lipids based on the fragments observed"
- [methods] Identification level assignment depends on fatty acyl constituents vs class vs precursor: "they are assigned a number based on whether they are identified by class and fatty acyl constituents (1), by data-independent analysis (2), only by class (3), or only by precursor m/z without"
- [methods] Fragment intensity/scan thresholding and ranking rule: "Each fragment is assigned 1 if it is above the user defined minimum intensity and scans threshold and 0 if the fragment does not meet these criteria or was not found within the m/z tolerance"
- [abstract] Ranking by summed fragment intensity: "Another unique feature of LipidMatch is ranking lipid identifications for a given feature by the sum of fragment intensities."
- [readme] README: scope of validated instruments and acquisition modes: "tested and validated using Q-Exactive orbitrap UHPLC-HRMS/MS data obtained from multiple sample types using targeted, data-dependent top-N (ddMS2-topN), and all ion fragmentation (AIF) approaches"
- [readme] README: Waters limitation: "The software does not currently support Waters files."
