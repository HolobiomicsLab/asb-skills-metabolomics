---
name: workflow-runtime-benchmarking
description: Use when a ddMS2 centroided mzML sample (~4000 spectra) must be timed
  through the full PFΔScreen pipeline — 'Run FeatureFinding' followed by 'Run PFASPrioritization'
  — to verify the sub-1-minute runtime claim.
when_to_use_negative:
- Input is already a custom/vendor feature list (external_feature_list.xlsx route)
  — the benchmark should exercise the full pyOpenMS feature-finding path to reflect
  the reported whole-workflow timing.
- Input spectra are profile-mode or acquired without data-dependent MS2 — PFΔScreen
  requires centroided ddMS2 mzML, so timing such data would not validate the paper's
  runtime claim.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PFΔScreen
  role: Software under benchmark; its FeatureFinding and PFASPrioritization GUI steps
    are timed end-to-end
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Performs the feature-detection step whose execution time is included in the
    measured runtime
- name: OpenMS
  role: Supplies the metabolites (5% RMS) isotope model used in the timed feature-detection
    configuration
- name: MSConvert
  role: Converts vendor raw data (Agilent .d) into the mzML format consumed by the
    benchmark
- name: Python
  role: Runtime environment (Python 3.9 required); time.perf_counter used to time
    the workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:82bc4a3e61a2db15bf3d105cfb9f6459b66df837f33387bde12bfeaec7ac3b7a
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# workflow-runtime-benchmarking

## Summary

End-to-end runtime benchmark of the complete PFΔScreen workflow (pyOpenMS feature finding plus PFAS prioritization) on a ddMS2 mzML sample of ~4000 spectra, validating the paper's claim that runtimes are usually below 1 minute.

## When to use

Use when a ddMS2 centroided mzML sample (~4000 spectra) must be timed through the full PFΔScreen pipeline — 'Run FeatureFinding' followed by 'Run PFASPrioritization' — to verify the sub-1-minute runtime claim.

## When NOT to use

- Input is already a custom/vendor feature list (external_feature_list.xlsx route) — the benchmark should exercise the full pyOpenMS feature-finding path to reflect the reported whole-workflow timing.
- Input spectra are profile-mode or acquired without data-dependent MS2 — PFΔScreen requires centroided ddMS2 mzML, so timing such data would not validate the paper's runtime claim.

## Inputs

- ddMS2 mzML sample file with centroided spectra, approximately 4000 spectra (vendor-converted e.g. from Agilent .d via MSConvert)
- Optional: blank control mzML (left unused for the runtime benchmark)
- PFΔScreen parameter settings (feature finding, MS2 alignment, blank correction, prioritization thresholds)

## Outputs

- runtime_report containing elapsed seconds and a pass/fail flag against the 60 s threshold
- PFΔScreen results table (Excel) and interactive HTML plots (MD/C-m/C plot, m/z vs. RT plot, KMD vs. m/z, m/C histogram) as byproducts of the timed run

## How to apply

Install PFΔScreen in a Python 3.9 environment via 'pip install -r requirements.txt' (or the Installation.bat batch file), then launch the GUI with Run_PFAScreen.bat. Load the centroided ddMS2 sample mzML via 'Browse Sample.mzML' (blank mzML optional and unused for a pure runtime benchmark) and execute 'Run FeatureFinding' with pyOpenMS using the paper's parameters (10 ppm mass error, 2000-count MS1 intensity threshold, OpenMS metabolites 5% RMS isotope model, FWHM 1 s to 1 min, ≥2 isotopic traces). Next execute 'Run PFASPrioritization' covering MS2 alignment (5 mDa, 0.2 min RT), blank correction, MD/C-m/C reduction (m/C cutoff 30, MD/C < +0.003), KMD analysis with (CF2)n and CF2O units (±2 mDa, ≥3 homologues), diagnostic fragment detection (±2 mDa, 2000-count MS2 threshold), and suspect screening (4 mDa). Wrap both steps in Python's time.perf_counter (or an equivalent timer) and record elapsed seconds in seconds. Validation passes if total elapsed time is < 60 s for ~4000 spectra, consistent with the reported claim; write runtime_report with elapsed seconds and a pass/fail flag.

## Related tools

- **PFΔScreen** (Software under benchmark; its FeatureFinding and PFASPrioritization GUI steps are timed end-to-end) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Performs the feature-detection step whose execution time is included in the measured runtime)
- **OpenMS** (Supplies the metabolites (5% RMS) isotope model used in the timed feature-detection configuration)
- **MSConvert** (Converts vendor raw data (Agilent .d) into the mzML format consumed by the benchmark)
- **Python** (Runtime environment (Python 3.9 required); time.perf_counter used to time the workflow)

## Examples

```
python -c "import time; t0=time.perf_counter(); run_feature_finding(); run_pfas_prioritization(); print(time.perf_counter()-t0)"
```

## Evaluation signals

- Elapsed time measured by time.perf_counter around FeatureFinding + PFASPrioritization is below 60 s for a ~4000-spectrum sample.
- The measured runtime is recorded with a concrete numeric value in seconds, not a qualitative label.
- The timed run uses the paper's parameter set (10 ppm, 2000 counts, ±2 mDa KMD, ≥3 homologues, 4 mDa suspect screening), not defaults or arbitrary values.
- runtime_report contains both elapsed_seconds and a pass/fail flag derived from the 60 s threshold.
- The run completes the whole workflow, evidenced by the generated results table and HTML plots in the sample-named output folder.

## Limitations

- The runtime claim is reported without hardware or computing environment specifications, so benchmark results are machine-dependent and not directly comparable across systems.
- Runtimes are 'usually' below 1 min — samples with very different feature counts or spectra numbers may exceed the threshold.
- The short runtime is explicitly framed as enabling convenient parameter adjustment; timing on other sample types (e.g., GC-HRMS data or external feature lists) is not covered by the claim.

## Evidence

- [results] Runtime claim reported without hardware specs: "PFΔScreen runtimes are usually below 1 min (e.g., for  ~ 4000 spectra per sample) for the whole workflow."
- [readme] README runtime corroboration: "The overall rather short runtime (e.g., less than one minute in case of 4000 spectra per sample for the whole workflow), allows a convenient adjustment of input parameters."
- [results] Feature finding parameters that must be set before timing: "The mass error for feature detection was set to 10 ppm, the MS1 intensity threshold was set to 2000 counts and the metabolites (5% RMS) isotope model from OpenMS was used"
- [readme] Installation requirement for the benchmark environment: "pip install -r requirements.txt"
- [readme] Input data requirements for the timed run: "Sample and blank for raw data input in PFΔScreen should have been measured under data-dependent acquisition (ddMS2) with centroided spectra"
