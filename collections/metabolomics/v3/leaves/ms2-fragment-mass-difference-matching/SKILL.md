---
name: ms2-fragment-mass-difference-matching
description: Use when centroided ddMS2 mzML data (ideally one collision energy per
  precursor) and a completed FeatureFinding feature list are available, and features
  must be separated into PFAS-like analytes vs. matrix components using MS2 evidence.
when_to_use_negative:
- Only MS1 data are available (no ddMS2 mzML) — fragment matching would silently miss
  all MS2 evidence; use MS1-only prioritization instead
- Raw data are profile-mode or not acquired with data-dependent MS2 — PFΔScreen expects
  ddMS2 with centroided spectra
- The goal is full compound identification/quantification rather than feature prioritization
  — matching flags PFAS-like features but does not replace annotation with authentic
  reference standards
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_0121
tools:
- name: PFΔScreen
  role: Performs the MS2 fragment mass-difference and diagnostic-fragment matching
    within the PFASPrioritization tab and highlights hits in the RawDataVisualization
    MS2 extractor
  repo: https://github.com/JonZwe/PFAScreen
- name: Python
  role: Implementation language of PFΔScreen (with pyOpenMS for feature detection
    feeding the MS2 matching step)
  repo: https://github.com/JonZwe/PFAScreen
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:42280a1c9ee1c335f58ed8464c37d0494009d36e7a549ea94fde86dbdbb76cc3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - PFΔScreen
  - Python
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# MS2 fragment mass difference matching

## Summary

PFΔScreen's MS2-based prioritization step that matches fragment mass differences and polarity-dependent diagnostic fragments against a built-in ~900-fragment literature list to flag PFAS-like features in non-target HRMS data.

## When to use

Use when centroided ddMS2 mzML data (ideally one collision energy per precursor) and a completed FeatureFinding feature list are available, and features must be separated into PFAS-like analytes vs. matrix components using MS2 evidence.

## When NOT to use

- Only MS1 data are available (no ddMS2 mzML) — fragment matching would silently miss all MS2 evidence; use MS1-only prioritization instead
- Raw data are profile-mode or not acquired with data-dependent MS2 — PFΔScreen expects ddMS2 with centroided spectra
- The goal is full compound identification/quantification rather than feature prioritization — matching flags PFAS-like features but does not replace annotation with authentic reference standards

## Inputs

- Centroided ddMS2 mzML file of the sample (vendor-independent, generated via MSConvert; ideally one collision energy per precursor)
- Sample feature list from the completed FeatureFinding tab (or preprocessed external_feature_list.xlsx)
- Optional blank feature list for blank-corrected features
- Built-in polarity-dependent diagnostic-fragment list (~900 literature-derived fragments)

## Outputs

- Prioritized feature table (Excel) with fragment and diagnostic-fragment hit fields per feature
- MS2 spectra in RawDataVisualization with highlighted fragment mass differences and diagnostic fragments where detected
- Interactive HTML plots (MD/C-m/C plot, m/z vs. RT with/without MS2 raw data, KMD vs. m/z, m/C histogram) in a folder named after the sample

## How to apply

Load the FeatureFinding-tab output (sample feature list, optionally the blank feature list) together with the corresponding centroided ddMS2 mzML file into PFΔScreen, since MS2-based evaluation requires the raw data. Exhaustively match fragment mass differences and diagnostic fragments per polarity against the built-in diagnostic-fragment list (≈900 literature-derived fragments) at a mass tolerance of ±2 mDa and an MS2 intensity threshold of 2000 counts, as used for the M1 soil extract in the first iteration. Combine these MS2 hits with the other PFΔScreen prioritization outputs (MD/C-m/C reduction with m/C cutoff 30 and MD/C cutoff < +0.003, KMD analysis with (CF2)n/CF2O repeating units, and 4 mDa suspect screening) to build the reduced prioritized feature table. Rationale: PFAS classes share characteristic neutral losses and diagnostic fragments, so MS2 mass-difference evidence discriminates PFAS from matrix components that only satisfy MS1-based filters. The matched fragments and differences are rendered by RawDataVisualization with highlights so the analyst can verify each assignment.

## Related tools

- **PFΔScreen** (Performs the MS2 fragment mass-difference and diagnostic-fragment matching within the PFASPrioritization tab and highlights hits in the RawDataVisualization MS2 extractor) — https://github.com/JonZwe/PFAScreen
- **Python** (Implementation language of PFΔScreen (with pyOpenMS for feature detection feeding the MS2 matching step)) — https://github.com/JonZwe/PFAScreen

## Examples

```
Run_PFAScreen.bat  # then in the GUI: Browse Sample.mzML -> Run FeatureFinding -> Run PFASPrioritization
```

## Evaluation signals

- The exported Excel results table contains fragment and diagnostic-fragment hit columns populated for flagged features
- MS2 spectra rendered by RawDataVisualization show highlighted fragment mass differences and diagnostic fragments where detected
- Fragment/DF matches were obtained at a mass tolerance of ±2 mDa and an MS2 intensity threshold of 2000 counts (per-polarity matching)
- Features flagged by MS2 matching are consistent with the MD/C-m/C and KMD outputs (homologous series with ≥3 homologues at ±2 mDa) rather than isolated false positives
- Runtime stays in the expected range (whole workflow under ~1 min for ~4000 spectra per sample)

## Limitations

- MS2 coverage is incomplete: in the M1 soil extract only 1026 of 2450 acquired MS2 spectra corresponded to detected features (417 unique, ~11% coverage in the first iteration), so features lacking MS2 cannot be evaluated by this step
- The built-in diagnostic-fragment list is literature-derived and polarity-dependent; unknown PFAS classes may lack listed fragments
- Parameters are sample- and instrument-dependent and must be adjusted by the end user; no universal defaults are provided
- Data evaluation only works when the corresponding mzML files are given alongside feature lists; otherwise MS2 data would be missing

## Evidence

- [results] Fragment/DF detection at ±2 mDa and 2000 counts: "at a mass tolerance set to ± 2 mDa and an MS2 intensity threshold of 2000 counts in the M1 soil extract (first iteration)"
- [readme] PFΔScreen uses fragment mass differences and diagnostic fragments in MS2 data: "PFΔScreen uses several techniques for prioritization such as the MD/C-m/C approach, Kendrick mass defect (KMD) analysis and fragment mass differences and diagnostic fragments in the MS2 data."
- [readme] Highlighted hits in RawDataVisualization: "Afterwards, MS2 spectra displayed by the RawDataVisualization tool (MS2 extractor), have highlighted fragment mass differences and diagnostic fragments, if some were detected."
- [abstract] Prioritization combines MS2 diagnostic fragments with other techniques: "to perform efficient feature prioritization using several PFAS-specific techniques such as the highly promising MD/C-m/C approach, Kendrick mass defect analysis, diagnostic fragments (MS 2 ),"
- [results] MS2 coverage limitation in the M1 soil extract: "A total of 1026 out of 2450 acquired MS2 spectra corresponded to detected features, from which 417 unique spectra remained (~ 11% MS2 coverage in first iteration)."
