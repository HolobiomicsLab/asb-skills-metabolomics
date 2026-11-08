---
name: mass-spectrometry-tolerance-configuration
description: Decision support for setting mass and RT matching tolerances when linking
  peak-picked features to in silico lipid precursor and fragment libraries in LipidMatch.
when_to_use_negative:
- The data are low-resolution MS/MS spectra where ppm-level fragment matching is not
  meaningful and library matching relies on nominal-mass or spectral-similarity approaches
  instead.
- You need to perform the matching itself rather than configure tolerances — e.g.
  running the full rule-based identification workflow is a separate skill; this skill
  only covers choosing the tolerance parameters.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: LipidMatch
  role: R-based tool whose dialogue boxes expose the user-defined m/z tolerance (Da)
    for MS1 matching, the retention-time and m/z tolerances for associating .ms2 scans
    with features, and the ppm tolerance for fragment matching
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MSConvert
  role: Produces the peak-picked .ms2 files whose scan retention times and m/z values
    are matched to features within the configured RT/m/z tolerances
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:e62fbe12268bfc738714298a367b663f0daa11fa5d77dab0fdc0452176b76b7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - LipidMatch
  - MSConvert
  - MZmine
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# mass-spectrometry-tolerance-configuration

## Summary

Configure the m/z (Da), ppm, and retention-time tolerances that LipidMatch uses for MS1 precursor matching and MS/MS fragment matching when annotating lipid species from UHPLC-HRMS/MS feature tables.

## When to use

Decision support for setting mass and RT matching tolerances when linking peak-picked features to in silico lipid precursor and fragment libraries in LipidMatch.

## When NOT to use

- The data are low-resolution MS/MS spectra where ppm-level fragment matching is not meaningful and library matching relies on nominal-mass or spectral-similarity approaches instead.
- You need to perform the matching itself rather than configure tolerances — e.g. running the full rule-based identification workflow is a separate skill; this skill only covers choosing the tolerance parameters.

## Inputs

- Feature table CSV (e.g. from MZmine, XCMS, MS-DIAL, or Compound Discoverer) containing feature m/z, retention time, and intensities
- LipidMatch in silico precursor and fragment libraries in .csv format
- MS/MS peak-picked spectra converted to .ms2 format via MSConvert

## Outputs

- Feature-candidate match table linking each feature's m/z to library lipid species and adducts within the MS1 Da tolerance
- Per-fragment match assignments (0/1) based on ppm m/z tolerance plus intensity and scans thresholds
- Ranked lipid identifications per feature ordered by summed fragment intensity

## How to apply

In LipidMatch, Step 1 performs exact mass matching at the MS1 level between in silico precursor ions and each feature's m/z using a user-defined m/z tolerance expressed in Da, so set this parameter according to your instrument's mass accuracy (e.g. orbitrap vs Q-TOF). Step 2 then finds MS/MS scans from .ms2 files within a user-defined retention time and m/z tolerance of each feature, so the RT window must be wide enough to capture scans acquired around the feature's chromatographic peak. Step 3 matches experimental fragments against in silico lipid fragment m/z values using a tolerance window in ppm, which is the appropriate unit for high-resolution fragment matching because ppm error is roughly constant across the mass range, whereas Da tolerance is used for the precursor-level match. The user also configures minimum intensity and minimum scans thresholds (Step 4) that gate whether a matched fragment is assigned 1 (usable for confirmation) or 0. Tolerances should be chosen so that dimers, multimers, and in-source fragments are excluded from precursor matches while true isobaric candidates are retained for downstream rule-based fragment confirmation.

## Related tools

- **LipidMatch** (R-based tool whose dialogue boxes expose the user-defined m/z tolerance (Da) for MS1 matching, the retention-time and m/z tolerances for associating .ms2 scans with features, and the ppm tolerance for fragment matching) — https://github.com/GarrettLab-UF/LipidMatch
- **MSConvert** (Produces the peak-picked .ms2 files whose scan retention times and m/z values are matched to features within the configured RT/m/z tolerances)

## Evaluation signals

- Precursor matches exclude dimers, multimers, and in-source fragments while retaining expected isobaric lipid candidates within the Da tolerance.
- For each feature, the set of associated MS/MS scans is non-empty and falls within the configured retention-time and m/z tolerance window.
- Fragment matches lie within the specified ppm tolerance of in silico fragment exact masses, with no systematic mass-error bias across the m/z range.
- Fragments assigned 1 satisfy the configured minimum intensity and minimum scans thresholds; fragments failing either criterion or outside the ppm window are assigned 0.
- Lipid identifications carry the correct annotation level (1–4) consistent with which necessary fragments passed the tolerance and threshold filters.

## Limitations

- Tolerance units differ by step: Da for MS1 precursor matching, ppm for fragment matching — conflating them produces incorrect match rates.
- LipidMatch was tested only on Windows 7–10 with R 3.3.3, and does not support Waters vendor files.
- Wider tolerances increase false positives among isobaric lipid candidates; the ranking by summed fragment intensity mitigates but does not eliminate ambiguity.

## Evidence

- [methods] MS1 precursor matching uses user-defined m/z tolerance in Da: "LipidMatch performs exact mass matching at the MS1 level between in silico precursor ions and each features m/z using a user defined m/z tolerance (Da)"
- [methods] MS/MS scan association uses RT and m/z tolerance: "MS/MS scans from .ms2 files within a user defined retention time and m/z tolerance of each feature is determined"
- [methods] Fragment matching uses a ppm tolerance window: "experimental fragments are matched against in silico lipid fragments m/z using a tolerance window (ppm)"
- [methods] Fragments gated by intensity and scans thresholds: "Each fragment is assigned 1 if it is above the user defined minimum intensity and scans threshold and 0 if the fragment does not meet these criteria"
- [readme] README confirms tool role and modular workflow: "LipidMatch identifications are obtained by matching experimental fragment m/z values with simulated library m/z values using in-silico fragmentation libraries"
