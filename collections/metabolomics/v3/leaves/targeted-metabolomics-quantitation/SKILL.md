---
name: targeted-metabolomics-quantitation
description: Use when processing targeted or semitargeted metabolomics, lipidomics,
  or fluxomics injection sequences (.mzML) that require automated peak detection,
  calibration, and concentration quantitation instead of manual vendor-software processing.
when_to_use_negative:
- Input is already an untargeted full-scan feature table (this skill is for targeted/semi-targeted
  workflows with known transitions)
- The task is flux analysis rather than targeted quantitation — BFAIR's MFA module
  is the better entry point
- Data is not CE-/GC-/LC-MS(/MS) or HPLC raw data (e.g., already reduced to a feature
  table with no chromatograms)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: SmartPeak
  role: Main application encapsulating the full targeted quantitation workflow (peak
    picking, calibration, QC, quantitation, report)
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: OpenMS
  role: Underlying toolkit providing most workflow steps except Aggregating standards,
    Find peaks, Align, Apply quality control, and Write to file
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak GUI
  role: Interactive session setup, workflow editing, calibration inspection ('View
    | Calibrators'), and report export
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak CLI
  role: Headless execution of the same workflow, optionally inside the smartpeak-cli
    Docker container
  repo: https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- name: pyOpenMS
  role: Parsing and processing the FeatureDB report with per-feature metadata such
    as peak_area
  repo: https://pyopenms.readthedocs.io
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:d8aacf261ead40d8f33090679f6a0da50091ba1f6fcd9c7987c5b06467045af7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - OpenMS
  - SmartPeak
  - SmartPeak CLI
  - SmartPeak GUI
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# targeted-metabolomics-quantitation

## Summary

Automates the SmartPeak targeted quantitation workflow on CE-, GC- and LC-MS(/MS) and HPLC data, from peak picking and integration through non-linear Gaussian-Mixtures calibration curve optimization, feature QC, quantitation, and report writing, using the OpenMS toolkit.

## When to use

Use when processing targeted or semitargeted metabolomics, lipidomics, or fluxomics injection sequences (.mzML) that require automated peak detection, calibration, and concentration quantitation instead of manual vendor-software processing.

## When NOT to use

- Input is already an untargeted full-scan feature table (this skill is for targeted/semi-targeted workflows with known transitions)
- The task is flux analysis rather than targeted quantitation — BFAIR's MFA module is the better entry point
- Data is not CE-/GC-/LC-MS(/MS) or HPLC raw data (e.g., already reduced to a feature table with no chromatograms)

## Inputs

- Raw chromatographic injections in .mzML format
- Sequence file specifying the injection order and sample classes (6 types: standards, blanks, samples, etc.)
- Transitions, components, and component groups metadata (CSV)
- Quantitation method / quant-method definitions including IS scoring
- Workflow configuration (.csv) specifying steps such as LOAD_RAW_DATA, PICK_MRM_FEATURES, SELECT_FEATURES, FILTER_FEATURES_BACKGROUND_INTERFERENCES

## Outputs

- Per-feature peak annotations (peak_area, RT) in a FeatureDB / feature table
- Calibration curve model with fitted parameters and R-squared (reported range 0.98–1.00 for Gaussian-Mixtures)
- Quantitation report (concentration per analyte per injection)
- Internal-standard scoring metrics (COMPUTE_IS_METRICS)
- QC and integration statistics (e.g., 94.3% of 574 Spike-in features correctly integrated)

## How to apply

Build a session from a sequence file (e.g., Targeted_quantitation_with_LC-MSMS_5500_QTRAP_RapidRIP) and run the SmartPeak workflow: LOAD_RAW_DATA, MRM feature picking via PICK_MRM_FEATURES, SELECT_FEATURES, calibration curve fitting with the non-linear Gaussian-Mixtures method (which outperformed generic linear vendor methods with R-squared = 0.99), then accuracy, precision, recovery, blank, solvent, RT, feature QC, and report generation. Data must be organized into the 6 sample classes that SmartPeak assigns (e.g., standards, blanks, samples). Evaluate via the SmartPeak GUI 'View | Calibrators' for fitted model inspection, 'Actions | Integrity checks' for input validation, and 'View | features (table)' for per-feature peak_area review before exporting the report.

## Related tools

- **SmartPeak** (Main application encapsulating the full targeted quantitation workflow (peak picking, calibration, QC, quantitation, report)) — https://github.com/AutoFlowResearch/SmartPeak
- **OpenMS** (Underlying toolkit providing most workflow steps except Aggregating standards, Find peaks, Align, Apply quality control, and Write to file) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak GUI** (Interactive session setup, workflow editing, calibration inspection ('View | Calibrators'), and report export) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak CLI** (Headless execution of the same workflow, optionally inside the smartpeak-cli Docker container) — https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- **pyOpenMS** (Parsing and processing the FeatureDB report with per-feature metadata such as peak_area) — https://pyopenms.readthedocs.io

## Examples

```
docker run --rm -ti -v C:/data:/sample-data autoflowresearch/smartpeak-cli:latest bash
```

## Evaluation signals

- Integrity checks pass after loading the sequence (Actions | Integrity checks)
- Gaussian-Mixtures calibration achieves R-squared in the 0.98–1.00 range (paper benchmark: 0.99)
- Feature integration correctness >95% on user-optimized settings; paper reports 94.3% correct integration of 574 Spike-in features
- Per-feature peak_area values present for every transition in the exported FeatureDB report
- Calibrators view shows a fitted mixture model with selected calibration points and no flagged failures

## Limitations

- Processing is limited to 5000 feature-table entries and 9000 chromatogram points in the GUI views
- Workflow steps such as Aggregating standards, Find peaks, Align, Apply quality control, and Write to file are not supported by stock OpenMS and are SmartPeak-specific
- R-squared of 0.99 is dataset-specific (Targeted_quantitation_with_LC-MSMS_5500_QTRAP_RapidRIP); other matrices may fall outside the 0.98–1.00 validation range

## Evidence

- [results] SmartPeak automated reproducible quantitation and supported all required steps for targeted metabolomics, including peak picking, calibration, pooling, accuracy, precision, recovery, blank, solvent, RT, feature selection, aggregate and feature quality control, quantitation, and report writing: [evidence span withheld — anchor status: not_found]
- [results] Non-linear Gaussian-Mixtures peak picking, calibration, and integrated accuracy and precision algorithmic methods implemented in SmartPeak outperformed generic linear methods in targeted proteomics.: [evidence span withheld — anchor status: not_found]
- [readme] Start the session with ``File | Load session from sequence``: [evidence span withheld — anchor status: external_doc]
