---
name: mrm-feature-metavalue-extraction
description: Use when an MRM peak-picking workflow has produced OpenMS FeatureMaps
  and you need to flatten feature meta-values (peak_area, logSN, RT, quality) into
  a tidy per-compound CSV for internal-standard scoring or quantitation.
when_to_use_negative:
- Input is already a flat per-compound CSV (e.g. BFAIR-processed) rather than raw
  OpenMS FeatureMaps
- The experiment is non-targeted FIA-MS or DDA/DIA full-scan, where the MRM transition
  extraction and ComponentGroup structure do not apply
- You only need calibrated concentrations; calibration and report writing follow the
  same FeatureDB and are handled by separate steps
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SmartPeak
  role: Orchestrates the MRM workflow steps (PICK_MRM_FEATURES, SELECT_FEATURES, STORE_FEATURES,
    ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES) that produce the feature meta-values
    to be extracted
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: OpenMS
  role: Underlying toolkit whose FeatureMap data structures and meta-value handling
    are the source of the extracted values; SmartPeak implements the steps OpenMS
    lacks (aggregation, QC application, file writing)
- name: pyOpenMS
  role: Python binding used to parse and post-process the SmartPeak/OpenMS outputs
    (e.g. mzTab/featureXML) when flattening meta-values into a FeatureDB outside the
    GUI/CLI
  repo: https://pyopenms.readthedocs.io/en/latest/id_by_mz.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:9ece602b224a0ea2cc642c34ec4d06918310f6b2ed4a00e50e6a9924be3d1ecb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - OpenMS
  - SmartPeak
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# MRM feature meta-value extraction

## Summary

Extracts per-compound, per-transition, and per-component meta-values (peak areas, heights, RT, logSN, tailing, asymmetry) from the OpenMS FeatureMap produced by the targeted MRM workflow and serializes them to the FeatureDB.csv table that downstream scoring, calibration, and IS-validation steps consume.

## When to use

Use when an MRM peak-picking workflow has produced OpenMS FeatureMaps and you need to flatten feature meta-values (peak_area, logSN, RT, quality) into a tidy per-compound CSV for internal-standard scoring or quantitation.

## When NOT to use

- Input is already a flat per-compound CSV (e.g. BFAIR-processed) rather than raw OpenMS FeatureMaps
- The experiment is non-targeted FIA-MS or DDA/DIA full-scan, where the MRM transition extraction and ComponentGroup structure do not apply
- You only need calibrated concentrations; calibration and report writing follow the same FeatureDB and are handled by separate steps

## Inputs

- OpenMS FeatureMap (.featureXML) or mzML chromatograms from MRM transition extraction
- Transition list / component-group definition CSV with IS designations
- Injection sequence file with sample types
- Workflow step configuration (LOAD_RAW_DATA, EXTRACT_SPECTRA_WINDOWS, PICK_MRM_FEATURES, SELECT_FEATURES, STORE_FEATURES)

## Outputs

- FeatureDB.csv table with one row per component/injection and columns for sample type, peak_area, logSN, RT, and feature meta-values
- Background-interference estimation feature table
- IS score table (is_scores.csv) downstream

## How to apply

After SELECT_FEATURES completes in the SmartPeak targeted MRM workflow (e.g. the 5500 QTRAP RapidRIP tutorial), iterate over each injection's FeatureMap and read the OpenMS meta-values attached to each Feature and subordinate ComponentGroup/Transition. Store features with the STORE_FEATURES step, and emit background-corrected quantities via ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES / STORE_FEATURE_BACKGROUND_ESTIMATIONS before filtering. Flatten the meta-values into a single-row-per-component FeatureDB table with sample type and injection metadata retained so downstream VALIDATE_IS and COMPUTE_IS_METRICS steps can consume it. The article notes OpenMS supports most steps except aggregation, peak finding, alignment, QC application, and file writing, so the flattening and serialization code lives in SmartPeak or a pyOpenMS post-processing script; pyOpenMS can parse the mzML/featureXML outputs directly.

## Related tools

- **SmartPeak** (Orchestrates the MRM workflow steps (PICK_MRM_FEATURES, SELECT_FEATURES, STORE_FEATURES, ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES) that produce the feature meta-values to be extracted) — https://github.com/AutoFlowResearch/SmartPeak
- **OpenMS** (Underlying toolkit whose FeatureMap data structures and meta-value handling are the source of the extracted values; SmartPeak implements the steps OpenMS lacks (aggregation, QC application, file writing))
- **pyOpenMS** (Python binding used to parse and post-process the SmartPeak/OpenMS outputs (e.g. mzTab/featureXML) when flattening meta-values into a FeatureDB outside the GUI/CLI) — https://pyopenms.readthedocs.io/en/latest/id_by_mz.html

## Examples

```
docker run --rm -ti -v /data:/sample-data autoflowresearch/smartpeak-cli:latest bash -- run -w FeatureDB --path /sample-data/Targeted_quantitation_with_LC-MSMS_5500_QTRAP_RapidRIP
```

## Evaluation signals

- Every analyte component in the extracted table is paired with a designated internal standard and non-null peak_area, logSN, RT values
- Row count equals the number of injections × number of transitions/components selected by SELECT_FEATURES
- Background-interference estimates precede FILTER_FEATURES_BACKGROUND_INTERFERENCES so that dropped features are not silently missing from the DB
- Values match those shown in SmartPeak GUI feature table/heatmap views for the same session (View | features (table))
- Sample types and injection metadata are preserved so VALIDATE_IS can distinguish calibrators from unknowns

## Limitations

- Feature table viewing in SmartPeak GUI is capped at 5000 entries and chromatogram viewer at 9000 points, so very large sequences must be consumed via CSV/pyOpenMS instead
- The step is specific to targeted MRM acquisitions; FIA-MS full-scan and DDA datasets follow different SmartPeak example pipelines
- Changelog not found in the repository, so version-specific meta-value naming may vary

## Evidence

- [methods] Meta-value schema and downstream IS consumption: [evidence span withheld — anchor status: not_found]
- [results] OpenMS/SmartPeak division of labor for steps not covered upstream: [evidence span withheld — anchor status: not_found]
