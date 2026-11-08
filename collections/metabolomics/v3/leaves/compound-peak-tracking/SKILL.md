---
name: compound-peak-tracking
description: Use when you have mzML/mzXML/mzData files from untargeted MS-based metabolomics
  and a list of m/z and RT pairs (e.g. from a long-term QC pool like MetNexs) and
  need to track compound peaks across batches to expose RT or intensity drift.
when_to_use_negative:
- When the goal is postacquisition intensity-drift correction rather than peak tracking/detection
  — use QC-MXP's tools for correcting intensity drifts instead.
- When you have no user-defined m/z and RT pairs for the compounds of interest — module_TrackCmp
  requires such a list; it is not an untargeted feature-finder.
- When the input is still vendor raw data that has not been converted to mzML/mzXML/mzData
  — convert first via the ProteoWizard converter.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: QC4Metabolomics
  role: Hosts module_TrackCmp, which detects and tracks peaks of user-defined m/z
    and RT pairs in converted mzML files and records the resulting quality parameters.
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: ProteoWizard
  role: Converts vendor-specific raw data to mzML so that module_TrackCmp can perform
    peak detection on the files.
- name: Docker
  role: Provides an OS-agnostic environment in which the ProteoWizard converter runs
    to produce the mzML inputs for peak tracking.
- name: R
  role: Language of the QC4Metabolomics package in which module_TrackCmp is implemented
    and whose ecosystem of packages underpins the extraction.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:7a88d6f4570afbc89e6d5540cc86efb8aa35ad65fb6c59324108d23869d1474e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - ProteoWizard
  - QC4Metabolomics
  - R
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# compound-peak-tracking

## Summary

Detects and tracks chromatographic peaks for user-defined compounds across acquired files by matching against a user-defined list of m/z and retention time (RT) pairs, extracting observed m/z, retention time, intensity, and peak shape to monitor quality in untargeted MS-based metabolomics runs.

## When to use

Use when you have mzML/mzXML/mzData files from untargeted MS-based metabolomics and a list of m/z and RT pairs (e.g. from a long-term QC pool like MetNexs) and need to track compound peaks across batches to expose RT or intensity drift.

## When NOT to use

- When the goal is postacquisition intensity-drift correction rather than peak tracking/detection — use QC-MXP's tools for correcting intensity drifts instead.
- When you have no user-defined m/z and RT pairs for the compounds of interest — module_TrackCmp requires such a list; it is not an untargeted feature-finder.
- When the input is still vendor raw data that has not been converted to mzML/mzXML/mzData — convert first via the ProteoWizard converter.

## Inputs

- mzML, mzXML, or mzData files (converted from vendor raw data via ProteoWizard in Docker) discovered by module_Files
- User-defined list of m/z and retention time (RT) pairs (e.g. compounds in a long-term QC pool such as MetNexs)
- File metadata (instrument name, project name, ionization mode, sample name) extracted by module_File_info

## Outputs

- Per-file, per-compound peak detections with observed m/z, retention time, intensity, and peak shape stored in the QC4Metabolomics database
- Time-series tables for each tracked compound suitable for dashboard plotting and batch-wise RT deviation comparison

## How to apply

In QC4Metabolomics, module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs applied to newly acquired mzML (or vendor-converted mzXML/mzData) files. For each matched compound record the observed m/z, retention time, intensity, and peak shape, tagging each detection with its file, sample name, ionization mode, and batch (as resolved from module_File_info's extracted metadata). To quantify RT stability, group the RT deviations per compound by batch: within-batch deviations should normally be less than 0.05 min, while differences between batches can be at least twice as large, so batch-level grouping is essential to separate the two regimes. The per-compound table of RT/intensity/peak-shape values over time is what the web dashboard plots and what a downstream drift analysis (e.g. the tryptophan intensity decrease in the MetNexs long-term QC pool) relies on. Use module_Warner thresholds on the extracted parameters to alert when a tracked compound's RT or intensity leaves its acceptable range.

## Related tools

- **QC4Metabolomics** (Hosts module_TrackCmp, which detects and tracks peaks of user-defined m/z and RT pairs in converted mzML files and records the resulting quality parameters.) — https://github.com/stanstrup/QC4Metabolomics
- **ProteoWizard** (Converts vendor-specific raw data to mzML so that module_TrackCmp can perform peak detection on the files.)
- **Docker** (Provides an OS-agnostic environment in which the ProteoWizard converter runs to produce the mzML inputs for peak tracking.)
- **R** (Language of the QC4Metabolomics package in which module_TrackCmp is implemented and whose ecosystem of packages underpins the extraction.)

## Evaluation signals

- Within-batch RT deviations for compounds in the long-term QC pool are normally less than 0.05 min, while interbatch differences are at least twice as large.
- Every tracked compound appears in the output table for each batch in which its corresponding m/z/RT pair was expected, with observed m/z, RT, intensity, and peak shape populated.
- Each peak record is linked to the correct sample name, ionization mode, and batch as extracted by module_File_info from the filename/metadata.
- A genuine instrument problem (e.g. the 2022 calibration offset or the tryptophan intensity drop over three weeks in MetNexs long-term QC samples) is visible as a consistent shift in the tracked parameters.
- Files that were never converted to mzML (or equivalent) do not produce tracked peaks, confirming the pipeline's conversion dependency.

## Limitations

- Instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate, so peak tracking alone does not replace external alerting — use module_Warner thresholds to compensate.
- Tracking depends on accurate user-supplied m/z and RT pairs; a lost or degraded calibrant signal (e.g. the 2023 pump malfunction) can compromise calibration and thus RT matching.
- Ion suppression or enhancement affecting only certain peaks, nonobvious contaminants, or loss of calibration in a limited mass region can produce tracking artifacts that require retrospective inspection on the dashboard.

## Evidence

- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [abstract] extracting diagnostic information such as observed m/z, retention time, intensity and peak shape: "extracting diagnostic information such as observed m/z , retention time, intensity and peak shape"
- [results] in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [abstract] The Shiny app is available as open-source code at https://github.com/stanstrup/QC4Metabolomics.: "The Shiny app is available as open-source code at https://github.com/stanstrup/QC4Metabolomics."
- [methods] module_File_info extracts information (e.g., instrument name, project name, ionization mode and sample name): "module_File_info extracts information (e.g., instrument name, project name, ionization mode and sample name)"
