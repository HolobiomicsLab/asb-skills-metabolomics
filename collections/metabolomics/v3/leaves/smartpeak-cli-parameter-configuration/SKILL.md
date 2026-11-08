---
name: smartpeak-cli-parameter-configuration
description: Reference for when you need to configure SmartPeak CLI for a targeted/semi-targeted
  CE-, GC-, or LC-MS(/MS) metabolomics, lipidomics, or fluxomics workflow — specifically
  when assembling a session directory (sequence file, parameters.
when_to_use_negative:
- When the goal is fully untargeted de-novo feature discovery without targeted transitions/components
  — SmartPeak targets targeted and semi-targeted workflows.
- When inputs are already processed feature tables or mzTab rather than raw mzML chromatograms
  — SmartPeak automates from peak detection through QC reporting, not post-hoc table
  analysis.
- When you need vendor-specific proprietary acquisition-file parsing rather than mzML
  — SmartPeak works on OpenMS-based mzML inputs.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SmartPeak
  role: Core application being configured; encapsulates advanced algorithms for automated
    CE-, GC-, and LC-MS(/MS) data processing for targeted/semi-targeted metabolomics
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak CLI
  role: Command-line interface used to launch configured workflows; equivalent of
    the SmartPeak GUI
- name: OpenMS
  role: Underlying toolkit providing the algorithm implementations (peak picking,
    calibration, feature QC) that the parameters.csv values tune
- name: Docker
  role: Container image autoflowresearch/smartpeak-cli used to run the configured
    workflow reproducibly
  repo: https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- name: pyOpenMS
  role: Python package used to parse and post-process SmartPeak outputs such as mzTab
- name: BFAIR
  role: Downstream post-processing package for FIA-MS outputs produced by configured
    SmartPeak workflows
  repo: https://github.com/AutoFlowResearch/BFAIR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:25717b986016e43c984b6500368cf737fb9e823ffe4d1b3ce2e64d1d29c8b283
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - BFAIR
  - Docker
  - OpenMS
  - SmartPeak
  - SmartPeak CLI
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# smartpeak-cli-parameter-configuration

## Summary

Skill for configuring and parameterizing SmartPeak CLI runs for automated targeted metabolomics data processing — assembling session data, workflow step sequences, and per-command parameters (e.g. acquisition windows, mass resolution, binning) needed to reproduce end-to-end automated processing pipelines. This is the configuration backbone that makes SmartPeak's approximate 100-fold speed-up over manual processing reproducible in practice.

## When to use

Reference for when you need to configure SmartPeak CLI for a targeted/semi-targeted CE-, GC-, or LC-MS(/MS) metabolomics, lipidomics, or fluxomics workflow — specifically when assembling a session directory (sequence file, parameters.csv, sample groups, transitions/components, quant methods) and choosing which workflow commands (e.g. LOAD_RAW_DATA, PICK_MRM_FEATURES, FILTER_FEATURES_BACKGROUND_INTERFERENCES) to execute and with what parameter values.

## When NOT to use

- When the goal is fully untargeted de-novo feature discovery without targeted transitions/components — SmartPeak targets targeted and semi-targeted workflows.
- When inputs are already processed feature tables or mzTab rather than raw mzML chromatograms — SmartPeak automates from peak detection through QC reporting, not post-hoc table analysis.
- When you need vendor-specific proprietary acquisition-file parsing rather than mzML — SmartPeak works on OpenMS-based mzML inputs.

## Inputs

- Sequence file (.csv) listing injections and sample groups
- Raw chromatography data (.mzML) for CE-, GC-, or LC-MS(/MS) instruments
- parameters.csv with per-workflow-command settings (e.g. acquisition_start, acquisition_end, resolution, max_mz, bin_step)
- Transitions/component lists and quantitation methods for targeted workflows
- CHEMISTRY/HMDB mapping files for annotation
- Optional Docker image autoflowresearch/smartpeak-cli for containerized runs

## Outputs

- Picked and integrated features per injection (chromatographic peak areas, m/z, RT)
- Feature metavalues including internal standard (IS) scores and computed IS metrics
- Background-interference estimations and filtered feature tables
- Calibration curve fits and quantitation reports
- Feature DB and pivot-table style reports (e.g. featuredb, pivottable)
- Logs and validation/integrity-check reports
- mzTab-compatible outputs post-processable with pyOpenMS or BFAIR

## How to apply

1) Create a session directory containing the sequence file, the sample groups, the raw data (mzML), and a parameters.csv; SmartPeak requires the .mzML chromatograms plus configuration files that can be parsed and processed by pyOpenMS. 2) Decide the workflow step list: pick from the supported commands (e.g. LOAD_RAW_DATA, EXTRACT_SPECTRA_WINDOWS, PICK_MRM_FEATURES, SELECT_FEATURES, ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES, FILTER_FEATURES_BACKGROUND_INTERFERENCES, MERGE_INJECTIONS, STORE_FEATURES_SAMPLE_GROUP); SmartPeak extends OpenMS with steps OpenMS does not cover, such as aggregating standards, aligning, applying QC, and writing to file. 3) Populate parameters.csv with per-command settings that drive the underlying OpenMS algorithms — e.g. for FIA-MS full-scan workflows set acquisition_start, acquisition_end, resolution, max_mz, and bin_step (the FIAMS_FullScan_Unknowns example uses acquisition_start 0, acquisition_end 30, resolution 12000, max_mz 1500, bin_step 20). 4) Invoke SmartPeak CLI (or the Docker image autoflowresearch/smartpeak-cli) pointing at the session directory and run the workflow; SmartPeak automates peak detection/integration, calibration curve optimization, and quality-control reporting end-to-end. 5) Validate inputs up front — SmartPeak provides command-line-based user input validation, and initialization steps validate acquisition meta-values of transitions/components/component groups and feature filters/QCs. 6) Export results with the report option (e.g. featuredb, pivottable — case-insensitive) and inspect the outputs; the workflow is considered correct when the outputs can be post-processed (e.g. via BFAIR or pyOpenMS mzTab) and quantitation matches expected feature integration quality.

## Related tools

- **SmartPeak** (Core application being configured; encapsulates advanced algorithms for automated CE-, GC-, and LC-MS(/MS) data processing for targeted/semi-targeted metabolomics) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak CLI** (Command-line interface used to launch configured workflows; equivalent of the SmartPeak GUI)
- **OpenMS** (Underlying toolkit providing the algorithm implementations (peak picking, calibration, feature QC) that the parameters.csv values tune)
- **Docker** (Container image autoflowresearch/smartpeak-cli used to run the configured workflow reproducibly) — https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- **pyOpenMS** (Python package used to parse and post-process SmartPeak outputs such as mzTab)
- **BFAIR** (Downstream post-processing package for FIA-MS outputs produced by configured SmartPeak workflows) — https://github.com/AutoFlowResearch/BFAIR

## Examples

```
docker run --rm -ti -v /data:/sample-data autoflowresearch/smartpeak-cli --load-session /sample-data/FIAMS_FullScan_Unknowns --report featuredb pivottable
```

## Evaluation signals

- Integrity checks pass on the loaded session (SmartPeak 'Integrity checks' action) and validation steps (VALIDATE_TRANSITIONS, VALIDATE_COMPONENTS, VALIDATE_COMPONENT_GROUPS, VALIDATE_FEATURE_FILTERS, VALIDATE_FEATURE_QCS) complete without errors.
- The chosen parameter set reproduces the documented FIAMS_FullScan_Unknowns defaults where applicable: acquisition_start 0, acquisition_end 30, resolution 12000, max_mz 1500, bin_step 20.
- Workflow completes end-to-end and report export (e.g. featuredb / pivottable) produces non-empty, well-formed outputs parseable by pyOpenMS or BFAIR.
- Feature integration quality: correct integrations comparable to the article's reported rates (e.g. 94.3% for Spike-in, 88.3% for mouse brain); >95% achievable with user-defined optimization methods.
- Internal standard scoring and computed IS metrics are present for injections where the optional IS scoring step was enabled.

## Limitations

- SmartPeak does not cover some OpenMS gaps (aggregating standards, find peaks, align, apply QC, write to file) — those are implemented in SmartPeak itself, so behavior may differ from stock OpenMS pipelines.
- Feature table views in the GUI are limited to 5000 entries and chromatogram viewers to 9000 points for performance; very large datasets may require CLI or server (HPC) routes.
- Manual post-processing and time compared to automated processing — the article frames the gain as ~100-fold vs manual, implying manual comparison is the baseline and errors in manual handling (e.g. inaccurate peak picking or annotation) account for a small fraction of remaining incorrect integrations (~5%).
- Changelog information for the software is not available in the provided sources.

## Evidence

- [methods] Configuration via parameters.csv with explicit numeric settings: [evidence span withheld — anchor status: not_found]
- [results] SmartPeak fills in the workflow steps OpenMS lacks: [evidence span withheld — anchor status: not_found]
- [readme] GUI session loading from sequence file, using example data directories: [evidence span withheld — anchor status: not_found]
- [readme] CLI uses --load-session to point at the session directory: [evidence span withheld — anchor status: not_found]
- [readme] Report export options for outputs: [evidence span withheld — anchor status: not_found]
- [results] Correct feature integration rates reported for automated workflow: [evidence span withheld — anchor status: not_found]
