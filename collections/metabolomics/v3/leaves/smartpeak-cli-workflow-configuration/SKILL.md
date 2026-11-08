---
name: smartpeak-cli-workflow-configuration
description: Use when raw CE-, GC-, LC-MS(/MS) or HPLC data must be processed end-to-end
  (peak detection, feature selection/QC, calibration, quantitation) via a scripted
  SmartPeak CLI run with a custom step sequence and CSV report outputs.
when_to_use_negative:
- You only need to inspect or review picked features/chromatograms interactively —
  SmartPeak GUI provides visualization, feature tables, heatmaps, and calibration
  views that the CLI does not
- The input is already a parsed feature/peak table rather than raw mzML data — post-processing
  (e.g. computing correctly-integrated fractions) belongs to downstream pyOpenMS/pandas
  or BFAIR tools
- You need to scale to large HPC workloads — SmartPeak Server is the containerized
  remote server option for computationally demanding workflows
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SmartPeak CLI
  role: Command-line equivalent of SmartPeak GUI; used to load a session, configure
    the validated --workflow step sequence, and export featuredb/pivottable reports
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak
  role: Application encapsulating the peak detection, feature selection/QC, calibration,
    and quantitation algorithms the configured workflow orchestrates
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: Docker
  role: Containerized execution via autoflowresearch/smartpeak-cli:latest with the
    data directory mounted at /sample-data
  repo: https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- name: OpenMS
  role: Underlying toolkit providing the algorithmic basis for SmartPeak's workflow
    steps
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: pyOpenMS
  role: Python package used to parse the exported FeatureDB/PivotTable CSVs for downstream
    verification
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:adc723ce6e2e32704ee3652ebb567b6332640d44fd4fd5a24b1e216ce4d77d2b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - Docker
  - OpenMS
  - SmartPeak
  - SmartPeak CLI
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# smartpeak-cli-workflow-configuration

## Summary

Configure and execute a SmartPeak CLI workflow by composing case-insensitive, validated step names (e.g. PICK_MRM_FEATURES, SELECT_FEATURES, CALIBRATE_FEATURES) into an ordered --workflow sequence against a loaded session, then export FeatureDB and PivotTable reports. This chains peak picking, feature selection, and calibration into one integrated targeted-quantitation pipeline instead of isolated steps.

## When to use

Use when raw CE-, GC-, LC-MS(/MS) or HPLC data must be processed end-to-end (peak detection, feature selection/QC, calibration, quantitation) via a scripted SmartPeak CLI run with a custom step sequence and CSV report outputs.

## When NOT to use

- You only need to inspect or review picked features/chromatograms interactively — SmartPeak GUI provides visualization, feature tables, heatmaps, and calibration views that the CLI does not
- The input is already a parsed feature/peak table rather than raw mzML data — post-processing (e.g. computing correctly-integrated fractions) belongs to downstream pyOpenMS/pandas or BFAIR tools
- You need to scale to large HPC workloads — SmartPeak Server is the containerized remote server option for computationally demanding workflows

## Inputs

- SmartPeak session.db (session directory with workflow.csv, sequence.csv, and parameter/input CSV templates)
- Raw .mzML mass-spectrometry data (CE-, GC-, LC-MS/MS, or HPLC acquisitions)
- parameters.csv with user-defined optimized algorithm parameters (e.g. FIAMS resolution 12000, bin_step 20)
- CHEMISTRY folder with HMDB mapping/adduct files for FIA-MS annotation runs

## Outputs

- FeatureDB.csv report
- PivotTable.csv report
- CLI exit status 0 on valid workflow configuration

## How to apply

Load the session with --load-session (a session.db bundled with tutorial data such as Targeted_quantitation_with_LC-MSMS_5500_QTRAP_RapidRIP, or a FIAMS_FullScan_Unknowns session), either locally or inside the autoflowresearch/smartpeak-cli Docker container with the data volume mounted at /sample-data. Compose the step sequence with --workflow using validated, case-insensitive command names: targeted MRM runs use LOAD_RAW_DATA, MAP_CHROMATOGRAMS, EXTRACT_CHROMATOGRAM_WINDOWS, ZERO_CHROMATOGRAM_BASELINE, PICK_MRM_FEATURES, SELECT_FEATURES, CALIBRATE_FEATURES, while full-scan FIA-MS runs use LOAD_RAW_DATA, EXTRACT_SPECTRA_WINDOWS, MERGE_SPECTRA, PICK_MS1_FEATURES, SEARCH_ACCURATE_MASS, STORE_ANNOTATIONS, STORE_FEATURES, ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES, STORE_FEATURE_BACKGROUND_ESTIMATIONS, FILTER_FEATURES_BACKGROUND_INTERFERENCES, MERGE_FEATURES, MERGE_INJECTIONS, STORE_FEATURES_SAMPLE_GROUP. Invalid workflow values are rejected before execution and cause exit status 1, so validate the sequence up front. Apply user-defined optimized algorithm parameters from parameters.csv where needed (e.g. FIAMS acquisition_start 0, acquisition_end 30, resolution 12000, max_mz 1500, bin_step 20). Export with --report featuredb pivottable and set --reports-out-dir (SmartPeak creates the directory if missing) so FeatureDB.csv and PivotTable.csv land in a known location rather than the working directory.

## Related tools

- **SmartPeak CLI** (Command-line equivalent of SmartPeak GUI; used to load a session, configure the validated --workflow step sequence, and export featuredb/pivottable reports) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak** (Application encapsulating the peak detection, feature selection/QC, calibration, and quantitation algorithms the configured workflow orchestrates) — https://github.com/AutoFlowResearch/SmartPeak
- **Docker** (Containerized execution via autoflowresearch/smartpeak-cli:latest with the data directory mounted at /sample-data) — https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- **OpenMS** (Underlying toolkit providing the algorithmic basis for SmartPeak's workflow steps) — https://github.com/AutoFlowResearch/SmartPeak
- **pyOpenMS** (Python package used to parse the exported FeatureDB/PivotTable CSVs for downstream verification)

## Examples

```
docker run --rm -ti -v C:/data:/sample-data autoflowresearch/smartpeak-cli:latest SmartPeakCLI --load-session /sample-data/session.db --workflow "LOAD_RAW_DATA,PICK_MRM_FEATURES,SELECT_FEATURES,CALIBRATE_FEATURES" --report featuredb pivottable --reports-out-dir /sample-data/reports
```

## Evaluation signals

- CLI exits with status 0; invalid --workflow values are validated before execution and cause exit status 1
- Both FeatureDB.csv and PivotTable.csv exist in the specified --reports-out-dir and contain the expected sample/component columns
- The full step sequence (e.g. LOAD_RAW_DATA through CALIBRATE_FEATURES for MRM) completes in order without interruption
- For FIAMS_FullScan_Unknowns with user-defined parameters, parsed feature tables should show >95% correctly integrated peaks in both m/z and RT dimensions
- Logs appear in the default log path (~/.SmartPeak on Linux/macOS or the SMARTPEAK_LOGS directory) confirming workflow progress

## Limitations

- SmartPeak GUI result views are performance-limited: 5000 entries for the feature table and 9000 points in the chromatogram viewer
- The GUI's estimated remaining workflow time is only a rough, regularly updated estimation
- User-defined optimization methods still left about 5% of features not correctly integrated, due to factors such as inaccurate peak picking or annotation
- OpenMS alone supports most but not all workflow steps (e.g. aggregating standards, find peaks, align, apply quality control, write to file are implemented in SmartPeak)

## Evidence

- [methods] Specify the targeted MRM command sequence LOAD_RAW_DATA, MAP_CHROMATOGRAMS, EXTRACT_CHROMATOGRAM_WINDOWS, ZERO_CHROMATOGRAM_BASELINE, PICK_MRM_FEATURES, SELECT_FEATURES, CALIBRATE_FEATURES via the --workflow option (values are case-insensitive and validated before execution; invalid values cause exit status 1).: [evidence span withheld — anchor status: not_found]
- [methods] Export both report types with --report featuredb pivottable, setting --reports-out-dir to a chosen output directory (SmartPeak creates it if missing) so the FeatureDB and PivotTable CSVs are written there instead of the current working directory.: [evidence span withheld — anchor status: not_found]
