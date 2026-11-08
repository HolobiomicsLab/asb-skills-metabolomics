---
name: pfas-diagnostic-fragment-screening
description: Use when ddMS2 mzML data are available and features from non-target screening
  must be prioritized as potential PFAS using MS2 diagnostic fragments and fragment
  mass differences.
when_to_use_negative:
- No MS2 raw data (mzML) are available — data evaluation only works when corresponding
  mzML files are given, otherwise MS2 data would be missing
- The goal is targeted quantification of known analytes with authentic reference standards
  rather than non-target PFAS prioritization
- Feature detection has not yet been performed and no custom feature list exists —
  run the FeatureFinding tab first
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: PFΔScreen
  role: Python GUI tool that performs the MS2 diagnostic-fragment and fragment-mass-difference
    matching and exports the prioritized table and HTML plots
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Feature detection from vendor-independent mzML ddMS2 raw data feeding the
    prioritization step
- name: MSConvert
  role: Conversion of vendor raw data (e.g. Agilent .d) into mzML with centroided,
    data-dependent spectra for input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:9ba92d30d8ac57f572dfd6c677694b01aee947f8e15c3198ac3cf464b30b1399
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - MSConvert
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# pfas-diagnostic-fragment-screening

## Summary

Prioritize PFAS features in non-target HRMS data by exhaustively matching MS2 diagnostic fragments and fragment mass differences against a built-in polarity-dependent literature fragment list (~900 fragments) inside PFΔScreen, flagging features whose MS2 spectra carry PFAS-typical fragments or homologous mass differences.

## When to use

Use when ddMS2 mzML data are available and features from non-target screening must be prioritized as potential PFAS using MS2 diagnostic fragments and fragment mass differences.

## When NOT to use

- No MS2 raw data (mzML) are available — data evaluation only works when corresponding mzML files are given, otherwise MS2 data would be missing
- The goal is targeted quantification of known analytes with authentic reference standards rather than non-target PFAS prioritization
- Feature detection has not yet been performed and no custom feature list exists — run the FeatureFinding tab first

## Inputs

- Centroided ddMS2 mzML raw data file of the sample (converted e.g. from Agilent .d via MSConvert)
- Feature list from PFΔScreen FeatureFinding tab or external feature table (e.g. SampleFeatures.xlsx)
- Optional blank mzML / blank feature list for background removal
- Built-in polarity-dependent diagnostic-fragment list (~900 literature fragments)

## Outputs

- Prioritized PFAS feature results table (Excel) with fragment/diagnostic-fragment hit fields
- Interactive HTML MS2 visualizations with highlighted fragment mass differences and diagnostic fragments
- MD/C-m/C plot, m/z vs. RT plot, KMD vs. m/z plot, and m/C histogram in a sample-named folder

## How to apply

Load the FeatureFinding-tab output (sample feature list, optionally blank feature list) together with the corresponding centroided ddMS2 mzML file into PFΔScreen, since MS2 evaluation requires raw data. Run PFASPrioritization to exhaustively match MS2 fragment mass differences and diagnostic fragments, per polarity, against the built-in diagnostic-fragment list using a mass tolerance of ±2 mDa and an MS2 intensity threshold of 2000 counts. Features with detected fragment mass differences or diagnostic fragments are retained in the prioritized results table; RawDataVisualization highlights matched fragment mass differences and diagnostic fragments in the rendered MS2 spectra so hits can be inspected manually. Combine with MD/C-m/C reduction, KMD analysis (e.g., (CF2)n, CF2O repeating units), and accurate-mass suspect screening (4 mDa) for a full prioritization workflow.

## Related tools

- **PFΔScreen** (Python GUI tool that performs the MS2 diagnostic-fragment and fragment-mass-difference matching and exports the prioritized table and HTML plots) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Feature detection from vendor-independent mzML ddMS2 raw data feeding the prioritization step)
- **MSConvert** (Conversion of vendor raw data (e.g. Agilent .d) into mzML with centroided, data-dependent spectra for input)

## Examples

```
Double-click Run_PFAScreen.bat, load sample and ddMS2 mzML, run FeatureFinding, then set MS2 mass tolerance ±2 mDa and intensity threshold 2000 counts and press "Run PFASPrioritization".
```

## Evaluation signals

- MS2 spectra rendered by RawDataVisualization show highlighted fragment mass differences and diagnostic fragments where detected
- The exported Excel results table contains prioritization fields including fragment/diagnostic-fragment hits alongside MD/C, m/C, KMD series assignment, and suspect hits
- Fragment matching used the documented thresholds: mass tolerance ±2 mDa and MS2 intensity threshold of 2000 counts
- Results and all named HTML plots exist in the folder named after the sample

## Limitations

- Only ~11% MS2 coverage of features was achieved in the first iteration (417 unique spectra from 1026 MS2 spectra matched to 3750 blank-corrected features), so many features cannot be evaluated by MS2 evidence
- No default or recommended input parameter values are provided; parameters must be adjusted per sample, instrument, and end-user needs
- Diagnostic-fragment list is literature-derived and polarity-dependent; novel PFAS classes outside the list may be missed without complementary KMD or suspect screening

## Evidence

- [methods] Exhaustively match MS2 fragment mass differences and diagnostic fragments against the built-in diagnostic-fragment list: [evidence span withheld — anchor status: not_found]
- [results] MS2 intensity threshold and tolerance from the article: "at a mass tolerance set to ± 2 mDa and an MS2 intensity threshold of 2000 counts in the M1 soil extract (first iteration)"
- [readme] README statement that MS2 highlighting occurs after PFASPrioritization: "Afterwards, MS2 spectra displayed by the RawDataVisualization tool (MS2 extractor), have highlighted fragment mass differences and diagnostic fragments, if some were detected."
- [abstract] Abstract listing of prioritization techniques including diagnostic fragments: "to perform efficient feature prioritization using several PFAS-specific techniques such as the highly promising MD/C-m/C approach, Kendrick mass defect analysis, diagnostic fragments (MS 2 ),"
