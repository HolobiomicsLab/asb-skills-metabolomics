---
name: chromatographic-feature-metadata-extraction
description: Reference for extracting per-feature metadata such as peak_area and calibration/IS
  metavalues from FeatureDB reports in targeted metabolomics workflows before fitting
  calibration curves or computing quantitation.
when_to_use_negative:
- Raw chromatograms have not been picked/selected yet — run PICK_MRM_FEATURES/SELECT_FEATURES
  first instead of trying to read calibration metavalues
- The experiment is untargeted FIA-MS or DDA/DIA discovery data rather than targeted
  quantitation with internal standards
- Data is already a processed quantitation report and no per-feature chromatographic
  metadata needs re-extraction
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SmartPeak
  role: Runs the targeted CE-/GC-/LC-MS(/MS) workflow that picks and selects MRM features
    and stores per-feature metavalues
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak GUI
  role: Interactive inspection of features (table/line/heatmap/matrix), Calibrators,
    Quant Methods, Chromatograms, and Actions | Report export
- name: SmartPeak CLI
  role: Command-line equivalent of the GUI workflow for scripted feature-metadata
    extraction over a session directory
- name: pyOpenMS
  role: Parses the FeatureDB report in Python to pull per-feature metadata such as
    peak_area
- name: OpenMS
  role: Underlying toolkit; current OpenMS supports most workflow steps except Aggregating
    standards, Find peaks, Align, Apply quality control, Write to file, which SmartPeak
    adds
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:4311f66445513b68d299bdad67bb1fef94a8b4e432234513e099bdc8a3b9b4c6
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# chromatographic-feature-metadata-extraction

## Summary

Extract per-feature chromatographic metadata (peak areas, metavalues, calibration-fitted parameters) from targeted CE-/GC-/LC-MS(/MS) processing sessions using SmartPeak/pyOpenMS so that downstream calibration, quantitation, and QC can be computed reproducibly.

## When to use

Reference for extracting per-feature metadata such as peak_area and calibration/IS metavalues from FeatureDB reports in targeted metabolomics workflows before fitting calibration curves or computing quantitation.

## When NOT to use

- Raw chromatograms have not been picked/selected yet — run PICK_MRM_FEATURES/SELECT_FEATURES first instead of trying to read calibration metavalues
- The experiment is untargeted FIA-MS or DDA/DIA discovery data rather than targeted quantitation with internal standards
- Data is already a processed quantitation report and no per-feature chromatographic metadata needs re-extraction

## Inputs

- Sequence folder with .mzML raw files and a sequence file
- FeatureDB report with per-feature metadata such as peak_area
- Transition/component group configuration (.csv inputs reloadable via File | Import File)

## Outputs

- Per-feature metadata table (metavalues incl. peak_area, background-interference estimates)
- Analyte-to-IS peak-area ratios per sample class
- Fitted calibration model parameters and selected calibrator points
- Exported report of user-selected variables

## How to apply

Load the data processing session (e.g. the Targeted_quantitation_with_LC-MSMS_5500_QTRAP_RapidRIP example) in SmartPeak GUI or CLI, run the targeted workflow through PICK_MRM_FEATURES and SELECT_FEATURES, then export or parse the per-feature report; the FeatureDB report can be read with pyOpenMS to obtain per-feature metadata such as peak_area. Compute analyte-to-internal-standard peak area ratios for each sample class (SmartPeak assigns all session data to one or more of 6 sample classes) before calibration or accuracy/precision scoring. Inspect fitted calibration models and selected calibrator points with View | Calibrators / Quant Methods, and export the chosen variables via Actions | Report. Validate inputs first with Actions | Integrity checks and monitor progress via View | log.

## Related tools

- **SmartPeak** (Runs the targeted CE-/GC-/LC-MS(/MS) workflow that picks and selects MRM features and stores per-feature metavalues) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak GUI** (Interactive inspection of features (table/line/heatmap/matrix), Calibrators, Quant Methods, Chromatograms, and Actions | Report export)
- **SmartPeak CLI** (Command-line equivalent of the GUI workflow for scripted feature-metadata extraction over a session directory)
- **pyOpenMS** (Parses the FeatureDB report in Python to pull per-feature metadata such as peak_area)
- **OpenMS** (Underlying toolkit; current OpenMS supports most workflow steps except Aggregating standards, Find peaks, Align, Apply quality control, Write to file, which SmartPeak adds)

## Examples

```
docker run --rm -ti -v C:/data:/sample-data autoflowresearch/smartpeak-cli:latest bash
```

## Evaluation signals

- Integrity checks pass for every transition/component/component-group/session input before the workflow runs
- Every session sample is assigned to one of the 6 expected sample classes
- Feature metavalues include non-missing peak_area for picked MRM features, and analyte/IS ratios are finite across calibration levels
- Fitted calibration models achieve R-squared ≈ 0.99 (paper reports 0.99; expect 0.98–1.00)
- GUI views (features table limited to 5000 entries, chromatogram viewer limited to 9000 points) render without truncation errors on the chosen session

## Limitations

- GUI feature table and chromatogram viewers are performance-capped (5000 entries / 9000 points), so large sessions must be subset via the explorer panes
- OpenMS alone does not support Aggregating standards, Find peaks, Align, Apply quality control, or Write to file — these require SmartPeak
- Only about 5% of features were not correctly integrated even with user-defined optimization, often due to inaccurate peak picking or annotation
- Calibration is a relative quantitation technique, not absolute concentration measurement

## Evidence

- [readme] GUI report/export support: [evidence span withheld — anchor status: external_doc]
- [readme] Calibration fit inspection: "A detailed look at the calibration fitted model and selected points for the model can be seen with ``View | Calibrators``."
- [methods] FeatureDB parsing: "These files can be parsed and processed by the `pyOpenMS <https://pyopenms.readthedocs.io/en/latest/id_by_mz.html?highlight=mztab>`_ Python package"
- [methods] Workflow step sequence: "PICK_MRM_FEATURES
                    SELECT_FEATURES"
- [results] Outperformed linear methods: [evidence span withheld — anchor status: not_found]
