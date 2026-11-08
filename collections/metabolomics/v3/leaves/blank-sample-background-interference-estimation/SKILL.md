---
name: blank-sample-background-interference-estimation
description: Use when processing untargeted FIA-MS (e.g. FIAMS_FullScan_Unknowns)
  sequences where analyte signals may arise from blank/solvent carryover and you need
  to assess, quantify, and remove features whose signal intensity in blank injections
  exceeds a defined percentage threshold.
when_to_use_negative:
- Input data are targeted MRM/SRM acquisitions (e.g. Targeted_quantitation_with_LC-MSMS_5500_QTRAP_RapidRIP)
  rather than untargeted full-scan FIA-MS unknowns — the background interference estimation
  is designed for untargeted workflows where solvent/blank-derived features must be
  removed.
- No blank injections are available in the sequence; the average background interference
  cannot be computed and the filter has no reference signal to threshold against.
- Features have already been filtered (e.g. by a prior SELECT_FEATURES or FILTER_FEATURES_BACKGROUND_INTERFERENCES
  run) and a second background-interference pass would remove valid analytes rather
  than solvent-derived artefacts.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SmartPeak
  role: Application that encapsulates the ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES
    and FILTER_FEATURES_BACKGROUND_INTERFERENCES steps for estimating and filtering
    background interference in blank injections of untargeted FIA-MS data
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak CLI
  role: Command-line interface providing an equivalent of the SmartPeak GUI to run
    the background interference estimation/filtering workflow
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: OpenMS
  role: Underlying toolkit that SmartPeak encapsulates for feature handling in the
    background interference workflow
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:022ec36bb5b6d394c4be4a2dfa477422e0686c85e3c5265f473c735885d3acb5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - OpenMS
  - SmartPeak
  - SmartPeak CLI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# blank-sample-background-interference-estimation

## Summary

Estimates, per component feature, the background interference contributed by blank injections sharing the same sequence segment in FIAMS_FullScan_Unknowns FIA-MS data, then filters out features whose signal intensity in blanks exceeds a user-specified threshold using SmartPeak's ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES and FILTER_FEATURES_BACKGROUND_INTERFERENCES steps.

## When to use

Use when processing untargeted FIA-MS (e.g. FIAMS_FullScan_Unknowns) sequences where analyte signals may arise from blank/solvent carryover and you need to assess, quantify, and remove features whose signal intensity in blank injections exceeds a defined percentage threshold.

## When NOT to use

- Input data are targeted MRM/SRM acquisitions (e.g. Targeted_quantitation_with_LC-MSMS_5500_QTRAP_RapidRIP) rather than untargeted full-scan FIA-MS unknowns — the background interference estimation is designed for untargeted workflows where solvent/blank-derived features must be removed.
- No blank injections are available in the sequence; the average background interference cannot be computed and the filter has no reference signal to threshold against.
- Features have already been filtered (e.g. by a prior SELECT_FEATURES or FILTER_FEATURES_BACKGROUND_INTERFERENCES run) and a second background-interference pass would remove valid analytes rather than solvent-derived artefacts.

## Inputs

- FIAMS_FullScan_Unknowns example dataset (.mzML raw data plus SmartPeak sequence/configuration files) from the SmartPeak repository
- Blank injections sharing the same sequence_segment as the samples of interest
- User-specified component list defining which components/features to estimate background interference for
- User-specified percentage threshold for signal intensity in blanks used by the FILTER_FEATURES_BACKGROUND_INTERFERENCES step

## Outputs

- background_interference_estimates.csv — a single CSV table with, for every component feature, its background-interference estimate and the pass/fail filter decision
- Persisted background interference estimates via STORE_FEATURE_BACKGROUND_ESTIMATIONS for inspection and re-use

## How to apply

Load the FIAMS_FullScan_Unknowns example dataset from the AutoFlowResearch/SmartPeak repository and identify the blank injections sharing the same sequence_segment as the samples of interest. Run the ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES command in SmartPeak (CLI or GUI) to compute the average background interference for each user-specified component from those blank samples, and persist the estimates with STORE_FEATURE_BACKGROUND_ESTIMATIONS. Then run FILTER_FEATURES_BACKGROUND_INTERFERENCES, which removes analytes whose integrals were marked as 'analyte in solvent' or 'IS in solvent', or flagged as a 'signal to noise' or 'harmonic noise' feature-filter outlier (on/t or below/t), an 'intensity transition' outlier, or a 'signal to noise' or 'intensity transition' quality-control outlier. The filter's decision is driven by the percentage of each component's signal intensity found in the blanks relative to the user-specified threshold; export a single CSV table (background_interference_estimates.csv) containing each feature's background-interference estimate and its pass/fail filter decision for downstream inspection or reuse.

## Related tools

- **SmartPeak** (Application that encapsulates the ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES and FILTER_FEATURES_BACKGROUND_INTERFERENCES steps for estimating and filtering background interference in blank injections of untargeted FIA-MS data) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak CLI** (Command-line interface providing an equivalent of the SmartPeak GUI to run the background interference estimation/filtering workflow) — https://github.com/AutoFlowResearch/SmartPeak
- **OpenMS** (Underlying toolkit that SmartPeak encapsulates for feature handling in the background interference workflow) — https://github.com/OpenMS/OpenMS

## Examples

```
smartpeak-cli --workflow ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES,FILTER_FEATURES_BACKGROUND_INTERFERENCES --data ./FIAMS_FullScan_Unknowns --session session.json
```

## Evaluation signals

- Verify that background_interference_estimates.csv exists after the workflow and is a well-formed CSV.
- Check that the CSV contains a background-interference estimate column and a pass/fail decision for every component feature in the input sequence.
- Confirm the FILTER_FEATURES_BACKGROUND_INTERFERENCES step removed features whose blank-signal intensity percentage exceeds the user-specified threshold, and retained features below it.
- Cross-check that features marked as 'analyte in solvent', 'IS in solvent', or flagged as 'signal to noise', 'harmonic noise' (on/t or below/t), or 'intensity transition' outliers were removed by the filter.
- Validate that persisted background interference estimates from STORE_FEATURE_BACKGROUND_ESTIMATIONS match the values written to the exported CSV.

## Limitations

- The workflow assumes blank injections exist in the same sequence segment as the samples; if they are absent or misassigned, background interference estimates are unreliable or cannot be computed.
- The filter is threshold-based and only removes features whose blank-signal intensity percentage exceeds the user-specified cutoff — analytes with borderline blank contributions may be incorrectly retained or removed depending on threshold choice.
- The estimate/filter pipeline is demonstrated on the FIAMS_FullScan_Unknowns example dataset; applying it to other acquisition modes (e.g. targeted MRM or DDA) is not validated by the source workflow.

## Evidence

- [methods] The workflow includes an ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES step followed by FILTER_FEATURES_BACKGROUND_INTERFERENCES; the filter removes analytes whose integrals were marked as 'analyte in solvent' or 'IS in solvent', or as a 'signal to noise' or 'harmonic noise' feature-filter outlier (on/t or below/t), or an 'intensity transition' outlier, or a 'signal to noise' or 'intensity transition' quality-control outlier (off or above).: [evidence span withheld — anchor status: not_found]
- [methods] Run the ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES command to compute the average background interference for each user-specified component from the blank samples.: [evidence span withheld — anchor status: not_found]
- [methods] Run the FILTER_FEATURES_BACKGROUND_INTERFERENCES command, filtering components by the percentage of their signal intensity found in the blanks using the user-specified threshold.: [evidence span withheld — anchor status: not_found]
- [readme] SmartPeak is an application that encapsulates advanced algorithms to enable fast, accurate, and automated processing of CE-, GC- and LC-MS(/MS) data, and HPLC data for targeted and semi-targeted metabolomics, lipidomics, and fluxomics experiments. The software is based on the OpenMS toolkit.: "SmartPeak is an application that encapsulates advanced algorithms to enable fast, accurate, and automated processing of CE-, GC- and LC-MS(/MS) data, and HPLC data for targeted and semi-targeted"
- [readme] The collection of examples is located at src/example/data directory of the SmartPeak source code. The directory contains examples of different kinds of data in .mzML format and their corresponding configuration files.: [evidence span withheld — anchor status: external_doc]
- [methods] `FIAMS FullScan Unknowns <https://github.com/AutoFlowResearch/SmartPeak/tree/develop/src/examples/data/FIAMS_FullScan_Unknowns>`_: "`FIAMS FullScan Unknowns <https://github.com/AutoFlowResearch/SmartPeak/tree/develop/src/examples/data/FIAMS_FullScan_Unknowns>`_"
