---
name: pfas-feature-prioritization-workflow-execution
description: Use when you need to run and validate end-to-end PFΔScreen execution
  time on a centroided ddMS2 mzML sample (~4000 spectra) to reproduce the paper's
  sub-1-minute runtime claim.
when_to_use_negative:
- Input data are not centroided ddMS2 mzML with data-dependent acquisition (e.g.,
  profile-mode or DIA data) — PFΔScreen's feature finding and MS2 alignment assume
  centroided DDA spectra
- You only need a targeted quantification of known PFAS analytes (<50 analytes) rather
  than non-target feature prioritization — the paper argues target analysis is insufficient
  for unknowns but this skill is for NTS prioritization
- Custom feature lists are supplied without the corresponding mzML files — data evaluation
  fails because MS2 data would be missing
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PFΔScreen
  role: Primary GUI tool executing feature finding and PFAS-specific prioritization
    (MD/C-m/C, KMD, diagnostic fragments, suspect screening); runtime benchmark target
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Performs feature detection from vendor-independent mzML raw data (ddMS2) within
    PFΔScreen
- name: OpenMS
  role: Provides the metabolites (5% RMS) isotope model used to filter features with
    unusual isotopic-trace peak shapes
- name: MSConvert
  role: Converts raw Agilent .d MS data into vendor-independent mzML format prior
    to PFΔScreen execution
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:6de9bc095d0ab32edb43058273972c774c550825735ceaee98d12bd49348d4c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - MSConvert
  - OpenMS
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# PFAS feature prioritization workflow execution (PFΔScreen)

## Summary

Executes the full PFΔScreen workflow — pyOpenMS feature finding on ddMS2 mzML data followed by PFAS-specific prioritization (MD/C-m/C, KMD, diagnostic fragments, suspect screening) — and benchmarks that the whole workflow completes in under 1 minute for ~4000-spectrum samples.

## When to use

Use when you need to run and validate end-to-end PFΔScreen execution time on a centroided ddMS2 mzML sample (~4000 spectra) to reproduce the paper's sub-1-minute runtime claim.

## When NOT to use

- Input data are not centroided ddMS2 mzML with data-dependent acquisition (e.g., profile-mode or DIA data) — PFΔScreen's feature finding and MS2 alignment assume centroided DDA spectra
- You only need a targeted quantification of known PFAS analytes (<50 analytes) rather than non-target feature prioritization — the paper argues target analysis is insufficient for unknowns but this skill is for NTS prioritization
- Custom feature lists are supplied without the corresponding mzML files — data evaluation fails because MS2 data would be missing

## Inputs

- Sample mzML file (centroided ddMS2 spectra, ~4000 spectra per sample, ideally one collision energy per precursor)
- Optional blank-control mzML file for blank correction
- Optional external feature list (external_feature_list.xlsx) with corresponding mzML files if vendor feature finding is preferred

## Outputs

- runtime_report with elapsed seconds (measured via time.perf_counter wrapping FeatureFinding + PFASPrioritization) and a pass/fail flag against the 60 s threshold
- PFΔScreen results table (Excel format) and interactive HTML plots (MD/C-m/C plot, m/z vs. RT plot with and without MS2 raw data, KMD vs. m/z with linked m/z vs. RT plot, m/C histogram) saved in a sample-named folder

## How to apply

Install PFΔScreen in a Python 3.9 environment via 'pip install -r requirements.txt' or the provided Installation.bat/Run_PFAScreen.bat batch files. Load a centroided ddMS2 mzML sample in the GUI ('Browse Sample.mzML'), optionally a blank, then 'Run FeatureFinding' with the paper's parameters: 10 ppm mass error, MS1 intensity threshold 2000 counts, OpenMS metabolites (5% RMS) isotope model, and FWHM filter 1 s to 1 min with at least two isotopic traces. Next run 'Run PFASPrioritization' with MS2 alignment at 5 mDa mass tolerance and 0.2 min RT tolerance, blank correction (removal within 2 mDa / <0.1 min RT unless ≥5-fold more abundant in sample), MD/C-m/C data reduction (m/C cutoff 30, MD/C cutoff < +0.003), KMD analysis with (CF2)n and CF2O repeating units (±2 mDa, ≥3 homologues), diagnostic fragment/fragment mass difference detection (±2 mDa, MS2 intensity threshold 2000 counts), and suspect screening at 4 mDa. Wrap the FeatureFinding and PFASPrioritization execution in Python's time.perf_counter (or an equivalent timer) and record elapsed seconds. Validation passes when the measured end-to-end runtime is below 60 s, consistent with the paper's claim that 'PFΔScreen runtimes are usually below 1 min (e.g., for ~4000 spectra per sample) for the whole workflow'.

## Related tools

- **PFΔScreen** (Primary GUI tool executing feature finding and PFAS-specific prioritization (MD/C-m/C, KMD, diagnostic fragments, suspect screening); runtime benchmark target) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Performs feature detection from vendor-independent mzML raw data (ddMS2) within PFΔScreen)
- **OpenMS** (Provides the metabolites (5% RMS) isotope model used to filter features with unusual isotopic-trace peak shapes)
- **MSConvert** (Converts raw Agilent .d MS data into vendor-independent mzML format prior to PFΔScreen execution)

## Examples

```
pip install -r requirements.txt && python -c "import time; t0=time.perf_counter(); run_feature_finding(); run_pfas_prioritization(); print('elapsed', time.perf_counter()-t0, 's'); assert time.perf_counter()-t0 < 60"
```

## Evaluation signals

- Measured end-to-end runtime (time.perf_counter around FeatureFinding + PFASPrioritization) is below 60 s for a ~4000-spectrum ddMS2 mzML sample
- Feature-finding output respects the FWHM filter (above 1 s, below 1 min) and includes at least two isotopic traces per reported feature
- Blank-corrected feature count is reduced relative to raw detections (e.g., article reports 4209 features reduced to 3750 after blank correction in ESI− for soil M1)
- Results folder contains the Excel results table and the expected HTML plots (MD/C-m/C, m/z vs. RT with/without MS2, KMD vs. m/z linked plot, m/C histogram)
- MS2 spectra displayed in RawDataVisualization show highlighted fragment mass differences and diagnostic fragments only where they were detected at ±2 mDa and MS2 intensity ≥2000 counts

## Limitations

- Runtime claim (less than 1 min for ~4000 spectra) is reported without hardware or computing environment specifications, so benchmark results may vary on other machines
- PFΔScreen version number is not stated, complicating exact reproducibility
- Python 3.9 is required; other Python versions are not supported by the automated installation
- Data evaluation only works when the corresponding mzML files accompany external feature lists; otherwise MS2 data would be missing
- No quantitative benchmark or comparison against alternative NTS data-reduction tools is reported in the article

## Evidence

- [results] PFΔScreen runtimes are usually below 1 min (e.g., for  ~ 4000 spectra per sample) for the whole workflow.: "PFΔScreen runtimes are usually below 1 min (e.g., for  ~ 4000 spectra per sample) for the whole workflow."
- [results] The mass error for feature detection was set to 10 ppm, the MS1 intensity threshold was set to 2000 counts: "The mass error for feature detection was set to 10 ppm, the MS1 intensity threshold was set to 2000 counts and the metabolites (5% RMS) isotope model from OpenMS was used"
- [results] Peaks reported after feature detection have to have a full width at half maximum (FWHM) above 1 s and below 1 min: "Peaks reported after feature detection have to have a full width at half maximum (FWHM) above 1 s and below 1 min, and at least two isotopic traces."
- [readme] The overall rather short runtime (e.g., less than one minute in case of 4000 spectra per sample for the whole workflow): "The overall rather short runtime (e.g., less than one minute in case of 4000 spectra per sample for the whole workflow), allows a convenient adjustment of input parameters."
- [readme] Please note that Python 3.9 is required: "Sample and blank for raw data input in PFΔScreen should have been measured under data-dependent acquisition (ddMS2) with centroided spectra, ideally with one collision energy per precursor."
