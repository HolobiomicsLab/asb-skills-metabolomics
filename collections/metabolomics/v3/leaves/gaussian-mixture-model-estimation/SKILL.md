---
name: gaussian-mixture-model-estimation
description: Use when fitting a calibration curve from calibration-standard injections
  (per-feature peak_area from a FeatureDB/mzML report) for targeted LC-MS/MS quantitation
  where a generic linear fit is insufficient.
when_to_use_negative:
- Data are already quantified (feature table with concentrations) — calibration fitting
  is the wrong step; move to accuracy/precision or QC reporting.
- Experiment is untargeted FIA-MS full-scan unknown discovery, where the SmartPeak
  tutorial workflow is FIAMS_FullScan_Unknowns rather than calibration-based quantitation.
- A vendor tool's generic linear calibration is adequate and you only need peak picking,
  not curve optimization.
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: SmartPeak
  role: Encapsulates the Gaussian-Mixtures calibration step within the automated targeted-quantitation
    workflow (peak detection → calibration → quantitation → QC reporting)
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak CLI
  role: Headless runner for the workflow whose Calibration step fits the Gaussian-Mixtures
    curve, equivalent to the GUI application
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak GUI
  role: Interactive inspection of the fitted calibration via "View | Calibrators"
    and "View | Workflow settings | Quant Methods"
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: OpenMS
  role: Toolkit underlying SmartPeak; supports most workflow steps except Aggregating
    standards, Find peaks, Align, Apply quality control, and Write to file, which
    SmartPeak implements
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: pyOpenMS
  role: Python parsing of the FeatureDB report (per-feature peak_area etc.) to prepare
    calibration inputs outside the GUI
  repo: https://pyopenms.readthedocs.io/en/latest/id_by_mz.html?highlight=mztab
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:f42d224bcfa2bb2078c1e161eda2732347e20f420d3ef0301d059218e6897c08
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
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Gaussian Mixture Model Estimation

## Summary

Non-linear Gaussian-Mixtures estimation of the metabolite calibration curve (concentration vs. analyte-to-internal-standard response), as implemented in SmartPeak's calibration step for targeted metabolomics.

## When to use

Use when fitting a calibration curve from calibration-standard injections (per-feature peak_area from a FeatureDB/mzML report) for targeted LC-MS/MS quantitation where a generic linear fit is insufficient.

## When NOT to use

- Data are already quantified (feature table with concentrations) — calibration fitting is the wrong step; move to accuracy/precision or QC reporting.
- Experiment is untargeted FIA-MS full-scan unknown discovery, where the SmartPeak tutorial workflow is FIAMS_FullScan_Unknowns rather than calibration-based quantitation.
- A vendor tool's generic linear calibration is adequate and you only need peak picking, not curve optimization.

## Inputs

- Calibration-standard injections in .mzML plus a FeatureDB/feature report with per-feature metadata (e.g. peak_area)
- Sequence/session configuration files (SmartPeak GUI "Load session from sequence" or SmartPeak CLI sample-data directory)
- Internal-standard transitions for each analyte

## Outputs

- Fitted calibration curve parameters (Gaussian-Mixture component weights, means, variances) per transition/analyte
- Concentration estimates for unknowns using the fitted curve
- Calibration coefficient of determination (R-squared) reported per component
- Calibrators/quant-method tables viewable in the SmartPeak GUI or exportable report

## How to apply

In the SmartPeak targeted-quantitation workflow, load calibration-standard injections (standard sample type; datasets such as Targeted_quantitation_with_LC-MSMS_5500_QTRAP_RapidRIP or Targeted_quantitation_with_HPLC_data) and compute analyte-to-internal-standard peak area ratios as the response. Fit the non-linear Gaussian-Mixtures calibration model to the concentration-vs-response points via the Calibration/Quantitation workflow step (CALIBRATION/CALIBRATORS), recording the fitted mixture component parameters (weights, means, variances). Inspect the fitted model and the selected calibration points with "View | Calibrators" and the quant-method with "View | Workflow settings | Quant Methods"; a coefficient of determination near 0.99 (paper range 0.98–1.00) is expected. Rationale: SmartPeak's non-linear Gaussian-Mixtures peak picking, calibration, and integrated accuracy-and-precision methods outperformed generic linear vendor methods in targeted proteomics.

## Related tools

- **SmartPeak** (Encapsulates the Gaussian-Mixtures calibration step within the automated targeted-quantitation workflow (peak detection → calibration → quantitation → QC reporting)) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak CLI** (Headless runner for the workflow whose Calibration step fits the Gaussian-Mixtures curve, equivalent to the GUI application) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak GUI** (Interactive inspection of the fitted calibration via "View | Calibrators" and "View | Workflow settings | Quant Methods") — https://github.com/AutoFlowResearch/SmartPeak
- **OpenMS** (Toolkit underlying SmartPeak; supports most workflow steps except Aggregating standards, Find peaks, Align, Apply quality control, and Write to file, which SmartPeak implements) — https://github.com/AutoFlowResearch/SmartPeak
- **pyOpenMS** (Python parsing of the FeatureDB report (per-feature peak_area etc.) to prepare calibration inputs outside the GUI) — https://pyopenms.readthedocs.io/en/latest/id_by_mz.html?highlight=mztab

## Examples

```
docker run --rm -ti -v C:/data:/sample-data autoflowresearch/smartpeak-cli:latest bash   # then run the calibration workflow on Targeted_quantitation_with_LC-MSMS_5500_QTRAP_RapidRIP
```

## Evaluation signals

- Fitted model R-squared falls in the expected range 0.98–1.00 (paper reports 0.99 for the calibration).
- "View | Calibrators" shows the selected calibration points on the fitted Gaussian-Mixtures curve without large outlier residuals.
- All 6 sample classes are correctly assigned so calibration standards are distinguished from blanks/solvents/QCs.
- Estimated concentrations of Spike-in and QC samples are consistent with the vendor tool's linear method, since non-linear methods recovered substantial amounts of correct integrations.
- Report export includes per-component calibration parameters and concentration columns as expected.

## Limitations

- Gaussian-Mixtures calibration is a non-linear relative-quantitation technique: it applies to targeted quantitation workflows, not to untargeted discovery or DDA/DIA spectral library matching.
- Automated fitting does not guarantee correct integration: about 5% of features were not correctly integrated even after user-defined optimization, due to inaccurate peak picking or annotation.
- Calibration quality depends on correct sample-class assignment in the session; all data in a session are assigned to one or more of 6 sample classes.
- GUI review is resource-limited (feature tables capped at 5000 entries; chromatogram viewer at 9000 points), which constrains interactive inspection of large calibration runs.

## Evidence

- [results] Non-linear Gaussian-Mixtures calibration curve estimation rationale: [evidence span withheld — anchor status: not_found]
- [abstract] Calibration step in the automated workflow: [evidence span withheld — anchor status: not_found]
- [methods] Calibration is relative quantitation fitting: [evidence span withheld — anchor status: not_found]
