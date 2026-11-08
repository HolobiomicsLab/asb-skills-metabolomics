---
name: calibration-curve-quantitation
description: Use when targeted LC-MS/MS (e.g. QTRAP MRM) peak features have been picked
  and selected and absolute quantitation via calibration curve fitting is needed,
  with reports exported as featuredb/pivottable CSVs.
when_to_use_negative:
- Input is already a feature/quantitation table and no raw chromatogram peak picking
  or calibration is needed
- The experiment is untargeted (e.g. FIAMS_FullScan_Unknowns or DDA/DIA analysis)
  rather than targeted MRM quantitation with calibration standards
- Calibration standards or a quantitation method defining the calibration curve are
  absent from the sequence
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SmartPeak
  role: Runs the integrated targeted MRM workflow including PICK_MRM_FEATURES, SELECT_FEATURES,
    and CALIBRATE_FEATURES for calibration-curve-based quantitation
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak CLI
  role: Command-line equivalent of the SmartPeak GUI used to run the workflow with
    --load-session, --workflow, and --report featuredb pivottable options
  repo: https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- name: OpenMS
  role: Underlying toolkit; most workflow steps are OpenMS-supported, with steps like
    aggregating standards and calibration implemented natively in SmartPeak
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: Docker
  role: Container (autoflowresearch/smartpeak-cli) for running the CLI with the data
    volume mounted at /sample-data
  repo: https://hub.docker.com/r/autoflowresearch/smartpeak-cli
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:cbbf3c9c54cb71992aed7222461189eb7f1c71646543040bce50f1833ff7d129
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - Docker
  - OpenMS
  - SmartPeak
  - SmartPeak CLI
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# calibration-curve-quantitation

## Summary

Calibrate picked MRM features against calibration standards and produce quantitative reports (FeatureDB and PivotTable) using SmartPeak's targeted metabolomics workflow, which chains peak picking, feature selection, calibration, and quantitation into one integrated run.

## When to use

Use when targeted LC-MS/MS (e.g. QTRAP MRM) peak features have been picked and selected and absolute quantitation via calibration curve fitting is needed, with reports exported as featuredb/pivottable CSVs.

## When NOT to use

- Input is already a feature/quantitation table and no raw chromatogram peak picking or calibration is needed
- The experiment is untargeted (e.g. FIAMS_FullScan_Unknowns or DDA/DIA analysis) rather than targeted MRM quantitation with calibration standards
- Calibration standards or a quantitation method defining the calibration curve are absent from the sequence

## Inputs

- SmartPeak session.db (from the Targeted_quantitation_with_LC-MSMS_5500_QTRAP_RapidRIP tutorial in src/examples/data)
- sequence.csv listing sample injections including calibration standards
- workflow.csv specifying the command sequence (…PICK_MRM_FEATURES, SELECT_FEATURES, CALIBRATE_FEATURES)
- parameter and input CSV templates for MRM feature picking and quantitation methods
- raw chromatogram data in .mzML format

## Outputs

- FeatureDB.csv report (per-sample, per-component quantitative features)
- PivotTable.csv report (sample/component pivot of quantitation results)
- Calibration curve fits inspectable via View | Calibrators / Quant Methods in the GUI

## How to apply

Load the SmartPeak session (session.db with sequence.csv, workflow.csv, and parameter/input CSV templates, e.g. from the Targeted_quantitation_with_LC-MSMS_5500_QTRAP_RapidRIP tutorial dataset in src/examples/data) and run the command sequence LOAD_RAW_DATA, MAP_CHROMATOGRAMS, EXTRACT_CHROMATOGRAM_WINDOWS, ZERO_CHROMATOGRAM_BASELINE, PICK_MRM_FEATURES, SELECT_FEATURES, CALIBRATE_FEATURES — calibration must follow peak picking and feature selection because it operates on the selected features. SmartPeak automates 'calibration curve optimization' as part of the chain from peak detection and integration to quality control reporting, and its non-linear Gaussian-Mixtures calibration methods outperformed generic linear vendor methods in targeted proteomics. Export results with --report featuredb pivottable, directing the CSVs to a --reports-out-dir that SmartPeak creates if missing. Command values are case-insensitive but validated before execution; invalid workflow step names cause exit status 1.

## Related tools

- **SmartPeak** (Runs the integrated targeted MRM workflow including PICK_MRM_FEATURES, SELECT_FEATURES, and CALIBRATE_FEATURES for calibration-curve-based quantitation) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak CLI** (Command-line equivalent of the SmartPeak GUI used to run the workflow with --load-session, --workflow, and --report featuredb pivottable options) — https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- **OpenMS** (Underlying toolkit; most workflow steps are OpenMS-supported, with steps like aggregating standards and calibration implemented natively in SmartPeak) — https://github.com/AutoFlowResearch/SmartPeak
- **Docker** (Container (autoflowresearch/smartpeak-cli) for running the CLI with the data volume mounted at /sample-data) — https://hub.docker.com/r/autoflowresearch/smartpeak-cli

## Examples

```
smartpeak --load-session /sample-data/session.db --workflow LOAD_RAW_DATA,MAP_CHROMATOGRAMS,EXTRACT_CHROMATOGRAM_WINDOWS,ZERO_CHROMATOGRAM_BASELINE,PICK_MRM_FEATURES,SELECT_FEATURES,CALIBRATE_FEATURES --report featuredb pivottable --reports-out-dir /sample-data/reports
```

## Evaluation signals

- Workflow exits with status 0 (invalid workflow step names cause exit status 1)
- Both FeatureDB.csv and PivotTable.csv exist in the specified --reports-out-dir and contain the expected sample/component columns
- Selected calibrator points and the fitted calibration model are inspectable and sensible via View | Calibrators / Quant Methods in the GUI
- Calibration was executed after PICK_MRM_FEATURES and SELECT_FEATURES in the command sequence, as the chain requires

## Limitations

- Workflow command values are validated before execution; invalid values cause exit status 1
- Non-linear Gaussian-Mixtures calibration outperformed generic linear vendor methods, so purely linear calibration may mis-integrate or mis-quantify features
- Report directory must be specified via --reports-out-dir, otherwise CSVs are written to the current working directory

## Evidence

- [intro] The workflow automates all steps from peak detection and integration over calibration curve optimization, to quality control reporting.: "automates all steps from peak detection and integration over calibration curve optimization, to quality control reporting"
- [results] Non-linear calibration methods outperformed generic linear methods in targeted proteomics.: [evidence span withheld — anchor status: not_found]
- [methods] Calibration is one step of the targeted MRM chain.: "PICK_MRM_FEATURES
                    SELECT_FEATURES"
- [results] SmartPeak supported all required targeted metabolomics steps including peak picking and calibration.: [evidence span withheld — anchor status: not_found]
- [readme] SmartPeak is based on the OpenMS toolkit for MS data processing.: "The software is based on the OpenMS toolkit."
