---
name: timeline-diagnostics-visualization
description: Use when instrument QA requires plotting a time-line of QC pool diagnostics
  (m/z, RT, intensity, peak shape) to detect calibration drift, sensitivity loss or
  contamination, e.g. MetNexs long-term QC injections on a Waters Synapt.
when_to_use_negative:
- When the goal is postanalysis correction of intensity drifts (QC-MXP territory)
  rather than visualization and detection of quality problems
- When the input data are not time-ordered QC injections on a single instrument —
  this skill tracks a user-defined compound list per instrument, not cross-instrument
  or biological-sample comparisons
- When the user only needs a generic quality overview (QCScreen) rather than compound-specific
  time-line diagnostics
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3361
tools:
- name: QC4Metabolomics
  role: Framework providing module_TrackCmp for peak detection and diagnostic extraction
    from user-defined m/z/RT pairs, and module_Warner for threshold-based email warnings
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Web application front-end presenting the resulting time-lines and dashboard
    of quality parameters
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Deployment container including ProteoWizard for converting vendor raw data
    to mzML and running the QC4Metabolomics modules OS-agnostically
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Language in which the QC4Metabolomics package and its modules (module_TrackCmp,
    module_Warner, module_Files) are implemented
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:73d65c3ef6bdea409e2dad4d21fd7aafb0fe820bc4b916f6253f14bcba4de9d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# timeline-diagnostics-visualization

## Summary

Tracks a user-defined list of m/z and retention time pairs across successive QC injections on a single instrument and plots the resulting time-line of observed m/z, retention time, intensity and peak-shape diagnostics, flagging deviations beyond user-defined thresholds (e.g. >20 ppm in m/z).

## When to use

Use when instrument QA requires plotting a time-line of QC pool diagnostics (m/z, RT, intensity, peak shape) to detect calibration drift, sensitivity loss or contamination, e.g. MetNexs long-term QC injections on a Waters Synapt.

## When NOT to use

- When the goal is postanalysis correction of intensity drifts (QC-MXP territory) rather than visualization and detection of quality problems
- When the input data are not time-ordered QC injections on a single instrument — this skill tracks a user-defined compound list per instrument, not cross-instrument or biological-sample comparisons
- When the user only needs a generic quality overview (QCScreen) rather than compound-specific time-line diagnostics

## Inputs

- mzML files (converted from vendor raw data) for successive QC pool injections, e.g. MetNexs long-term QC pool
- User-defined list of target m/z and retention time pairs for the instrument being tracked
- User-defined threshold values for warning notifications (e.g. 20 ppm m/z deviation)

## Outputs

- Per-injection time-line of observed m/z, retention time, intensity and peak-shape diagnostics for each tracked compound
- Dashboard visualization of RT and m/z deviation data across all batches
- Warning emails triggered by module_Warner when a quality parameter exceeds a user-defined threshold
- Flagged entries where m/z deviation exceeds 20 ppm, indicating serious analytical problems

## How to apply

Convert vendor raw files to mzML and let QC4Metabolomics' module_Files pick them up, then run module_TrackCmp with the user-defined list of target m/z and retention time pairs for the instrument in question (here the Waters Synapt). module_TrackCmp detects peaks and extracts observed m/z, retention time, intensity and peak-shape diagnostics for each injection of the long-term QC pool (MetNexs). Plot these values as a function of time to reveal trends such as a small systematic calibration offset, an order-of-magnitude sensitivity drop in June 2022, or loss of the calibrant signal when the calibrant pump malfunctioned. Apply user-defined warning thresholds via module_Warner — for this pool, m/z deviations larger than 20 ppm indicate serious analytical problems that should be addressed promptly, while RT deviations normally stay under 0.05 min within a batch. Correct decision-making relies on reading the time-line retrospectively as well as in near-real time, so the dashboard view can spot problems that the instrument itself never alerts on.

## Related tools

- **QC4Metabolomics** (Framework providing module_TrackCmp for peak detection and diagnostic extraction from user-defined m/z/RT pairs, and module_Warner for threshold-based email warnings) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Web application front-end presenting the resulting time-lines and dashboard of quality parameters) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Deployment container including ProteoWizard for converting vendor raw data to mzML and running the QC4Metabolomics modules OS-agnostically) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Language in which the QC4Metabolomics package and its modules (module_TrackCmp, module_Warner, module_Files) are implemented)

## Examples

```
Rscript -e "QC4Metabolomics::module_TrackCmp(mzml_dir='network_drive/QC', targets=read.csv('targets_waters_synapt.csv'))"
```

## Evaluation signals

- Per-injection time-line of observed m/z values should reproduce deviations normally less than 0.05 min in RT within a batch, with between-batch differences at least twice as large
- Deviation flags should fire only when m/z deviation exceeds the 20 ppm threshold established for the MetNexs long-term QC pool on the Waters Synapt
- Time-line should reveal known historical events (e.g. May/June 2022 calibration offset, June 2022 order-of-magnitude sensitivity drop, end-2022 calibrant signal loss) when replayed on archived data
- module_Warner should send an email warning only when a quality parameter exceeds the user-defined threshold, not on every injection
- Dashboard should present tracked compounds' m/z, RT, intensity and peak shape for every new file added by module_Files without manual intervention

## Limitations

- Requires a curated user-defined list of m/z and RT pairs per instrument; the skill cannot discover tracked compounds automatically
- Threshold choices (e.g. the 20 ppm m/z deviation limit) are instrument- and pool-specific and must be re-evaluated per setup
- The instruments themselves typically do not alert the user or stop acquisition when calibration is inaccurate — a problem that persists if monitoring is not deployed
- Time-line diagnostics alone do not correct drift; a separate tool such as QC-MXP is needed for intensity drift correction
- module_TrackCmp depends on module_Files successfully detecting and registering newly acquired mzML files; a failure in file ingestion stalls the time-line

## Evidence

- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [methods] module_Warner can send emails warning when a quality parameter exceeds a user defined threshold: "module_Warner can send emails warning when a quality parameter exceeds a user defined threshold"
- [results] RT and m/z deviation data for our long-term QC pool (MetNexs) analyzed within all batches: "RT and m/z deviation data for our long-term QC pool (MetNexs) analyzed within all batches"
- [results] in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [results] The instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate: "The instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate"
- [readme] QC4Metabolomics is available as code at https://github.com/stanstrup/QC4Metabolomics: "Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/"
