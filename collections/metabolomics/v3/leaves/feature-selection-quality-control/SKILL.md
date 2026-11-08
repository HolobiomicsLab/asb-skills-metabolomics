---
name: feature-selection-quality-control
description: Use when targeted MRM/LC-MSMS or HPLC peak-picked features must be filtered,
  selected, and QC'd against feature-filter and feature-QC meta-values before calibration
  and quantitation in a SmartPeak workflow.
when_to_use_negative:
- Input is already a selected/validated feature table needing only calibration or
  report export (skip picking and selection, run CALIBRATE_FEATURES / reports directly).
- Data are untargeted (e.g., Non-targeted_FIA-MS_analysis_with_Thermo_Orbitrap or
  DDA/DIA analysis) with no MRM transition list, feature filters, or feature QC definitions
  to validate against.
- Only peak detection quality assessment is needed without any selection or QC gating
  — PICK_MRM_FEATURES alone suffices.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SmartPeak
  role: Executes the feature selection, feature QC, background-interference filtering,
    and internal-standard scoring steps in the targeted workflow
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak CLI
  role: Command-line entry point used to run the PICK_MRM_FEATURES / SELECT_FEATURES
    / feature-QC command sequence with validated workflow options
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: OpenMS
  role: Underlying toolkit providing MRM feature detection; missing steps such as
    feature selection QC, aggregating standards, and quality control are implemented
    in SmartPeak
  repo: https://github.com/AutoFlowResearch/SmartPeak
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:a9620770d4c4dff32add4ecfa6da8ab04c39bbad60a1d1adc57be1ce3b5ad709
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - Docker
  - OpenMS
  - SmartPeak
  - SmartPeak CLI
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# feature-selection-quality-control

## Summary

SmartPeak's feature selection and QC stage for targeted metabolomics: after MRM peak picking, picked features are filtered (e.g., background-interference estimation), validated against feature filter and feature QC meta-values, optionally scored against internal standards, and only surviving features are passed to calibration and quantitation.

## When to use

Use when targeted MRM/LC-MSMS or HPLC peak-picked features must be filtered, selected, and QC'd against feature-filter and feature-QC meta-values before calibration and quantitation in a SmartPeak workflow.

## When NOT to use

- Input is already a selected/validated feature table needing only calibration or report export (skip picking and selection, run CALIBRATE_FEATURES / reports directly).
- Data are untargeted (e.g., Non-targeted_FIA-MS_analysis_with_Thermo_Orbitrap or DDA/DIA analysis) with no MRM transition list, feature filters, or feature QC definitions to validate against.
- Only peak detection quality assessment is needed without any selection or QC gating — PICK_MRM_FEATURES alone suffices.

## Inputs

- Raw LC-MS/MS chromatogram data in mzML format (loaded via LOAD_RAW_DATA and mapped with MAP_CHROMATOGRAMS)
- Session database (session.db) from the SmartPeak tutorial sequence (e.g., Targeted_quantitation_with_LC-MSMS_5500_QTRAP_RapidRIP)
- Transition, component, and component-group meta-values (MRM transition list)
- Feature filter definitions (VALIDATE_FEATURE_FILTERS) and feature QC definitions (VALIDATE_FEATURE_QCS)
- Internal standard definitions (for VALIDATE_IS / COMPUTE_IS_METRICS, optional)

## Outputs

- Selected (QC-passed) MRM features with background-interference and QC meta-values attached
- FeatureDB report CSV containing per-sample/per-component selected features
- Features suitable for downstream CALIBRATE_FEATURES and PivotTable reporting

## How to apply

Chain PICK_MRM_FEATURES followed by SELECT_FEATURES in the SmartPeak workflow sequence; SmartPeak's workflow commands are case-insensitive and validated before execution (invalid values cause exit status 1). Between picking and selection, apply ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES and FILTER_FEATURES_BACKGROUND_INTERFERENCES to reject features with strong background interference, and run the validation steps VALIDATE_TRANSITIONS, VALIDATE_COMPONENTS, VALIDATE_COMPONENT_GROUPS, VALIDATE_FEATURE_FILTERS, and VALIDATE_FEATURE_QCS, which initialize the workflow by checking acquisition meta-values. Optionally include VALIDATE_IS and COMPUTE_IS_METRICS for internal standard scoring. Prefer user-defined (non-linear, e.g., Gaussian-Mixtures) optimization methods over generic vendor linear methods, since SmartPeak's non-linear peak picking, calibration, and integrated accuracy/precision methods outperformed linear methods and user-defined optimization reached >95% correctly integrated features on four datasets. Inspect and adjust feature-filter/QC thresholds before rerunning, since only selected features flow into CALIBRATE_FEATURES and the final reports.

## Related tools

- **SmartPeak** (Executes the feature selection, feature QC, background-interference filtering, and internal-standard scoring steps in the targeted workflow) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak CLI** (Command-line entry point used to run the PICK_MRM_FEATURES / SELECT_FEATURES / feature-QC command sequence with validated workflow options) — https://github.com/AutoFlowResearch/SmartPeak
- **OpenMS** (Underlying toolkit providing MRM feature detection; missing steps such as feature selection QC, aggregating standards, and quality control are implemented in SmartPeak) — https://github.com/AutoFlowResearch/SmartPeak

## Examples

```
smartpeak --load-session session.db --workflow PICK_MRM_FEATURES,SELECT_FEATURES,FILTER_FEATURES_BACKGROUND_INTERFERENCES,CALIBRATE_FEATURES --report featuredb pivottable --reports-out-dir reports
```

## Evaluation signals

- Workflow containing PICK_MRM_FEATURES and SELECT_FEATURES completes without exit status 1 (invalid workflow values are rejected before execution).
- With user-defined optimization methods, >95% of features are correctly integrated on comparable datasets; automated defaults recover ~88–95% of features (94.3% of 574 in Spike-in, 89.4% of 5,213 in yeast, 88.3% of 7,094 in mouse brain, 94.9% of 5,050 in serum).
- All acquisition meta-values pass the initialization validations (transitions, components, component groups, feature filters, feature QC) before the workflow runs.
- Background-interference estimates are stored (STORE_FEATURE_BACKGROUND_ESTIMATIONS) and features failing FILTER_FEATURES_BACKGROUND_INTERFERENCES are excluded from selection.
- Downstream CALIBRATE_FEATURES succeeds and the exported FeatureDB/PivotTable CSVs contain the expected sample and component columns.

## Limitations

- Even with user-defined optimization, roughly 5% of features may remain incorrectly integrated, often due to inaccurate peak picking or annotation.
- Generic linear (vendor) methods recover fewer correct integrations than SmartPeak's non-linear Gaussian-Mixtures methods.
- In the SmartPeak GUI, feature table views are limited to 5000 entries and chromatogram views to 9000 points for performance reasons, which constrains interactive QC review of large sequences.

## Evidence

- [results] SmartPeak automated reproducible quantitation and supported all required steps for targeted metabolomics: [evidence span withheld — anchor status: not_found]
- [results] Non-linear Gaussian-Mixtures peak picking, calibration, and integrated accuracy and precision algorithmic methods: [evidence span withheld — anchor status: not_found]
- [methods] The workflow is initialized by validating the meta-values of feature QC which involves: [evidence span withheld — anchor status: not_found]
- [results] Current versions of OpenMS support most steps except: Aggregating standards, Find peaks, Align, Apply quality control, Write to file.: [evidence span withheld — anchor status: not_found]
