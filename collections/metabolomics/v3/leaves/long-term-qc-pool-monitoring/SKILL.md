---
name: long-term-qc-pool-monitoring
description: Use when you have repeated injections of a long-term QC pool (mzML/mzXML
  from untargeted MS metabolomics) and need to detect slow m/z, RT, intensity or peak-shape
  drift over batches, e.g. flagging m/z deviations > 20 ppm.
when_to_use_negative:
- The goal is batch-effect intensity correction of biological samples rather than
  QC monitoring — QC-MXP-style drift correction, not QC-pool tracking, is what is
  needed
- The input is a single batch with no repeated long-term QC pool injections, so no
  long-term drift signal exists to monitor
- The analysis already has a final feature table and only needs generic QC statistics
  (e.g. QCScreen) rather than tracking specific compounds over time
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: R/Shiny system whose module_TrackCmp detects peaks for user-defined m/z/RT
    pairs across QC-pool injections; module_Files ingests new mzML files and module_Warner
    flags threshold breaches
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Language of the QC4Metabolomics package that implements the tracking and ppm-deviation
    computation over the long-term QC timeline
- name: Shiny
  role: Web-application front end presenting tracked m/z, RT, intensity and peak-shape
    timelines for the long-term QC pool
- name: Docker
  role: Deployment container for QC4Metabolomics and its ProteoWizard-based mzML conversion
    pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:215a17650b3c1107f04fd7dba2af73552871e29e71e5c07d8e29f84cb4200604
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# long-term-qc-pool-monitoring

## Summary

Retrospectively or continuously monitor a long-term pooled QC sample (e.g. the MetNexs pool) by tracking user-defined m/z and RT pairs across injections and flagging drift in observed m/z, retention time, intensity and peak shape — deviations larger than 20 ppm in m/z signal serious analytical problems.

## When to use

Use when you have repeated injections of a long-term QC pool (mzML/mzXML from untargeted MS metabolomics) and need to detect slow m/z, RT, intensity or peak-shape drift over batches, e.g. flagging m/z deviations > 20 ppm.

## When NOT to use

- The goal is batch-effect intensity correction of biological samples rather than QC monitoring — QC-MXP-style drift correction, not QC-pool tracking, is what is needed
- The input is a single batch with no repeated long-term QC pool injections, so no long-term drift signal exists to monitor
- The analysis already has a final feature table and only needs generic QC statistics (e.g. QCScreen) rather than tracking specific compounds over time

## Inputs

- Long-term pooled QC sample data files in mzML (also mzXML/mzData) converted from vendor raw data, e.g. the MetNexs long-term QC pool
- A user-defined list of target m/z and retention time (RT) pairs to track
- Instrument name / project name / ionization mode / sample name metadata extracted per file

## Outputs

- Per-injection timeline of observed m/z, retention time, intensity and peak shape for each tracked compound
- m/z deviation (ppm) per tracked compound per injection, with entries flagged when deviation > 20 ppm
- Web-dashboard tables/plots and, optionally, email warnings from module_Warner when thresholds are exceeded

## How to apply

Load the long-term QC pool mzML files (e.g. the MetNexs pool acquired on the Waters Synapt) and register them via module_Files/module_File_info so each injection carries instrument, project and sample metadata. Run module_TrackCmp on a user-defined list of target m/z and retention time pairs (run separately per instrument, e.g. Waters Synapt vs other instruments) to detect peaks and extract observed m/z, RT, intensity and peak shape diagnostics per injection. Compute m/z deviations in ppm from the target m/z across the timeline, and flag injections where deviation exceeds 20 ppm — the article's long-term QC tracking shows deviations larger than 20 ppm indicate serious analytical problems to address promptly. Optionally wire module_Warner to email when a tracked quality parameter crosses the threshold, and use module_Productivity/module_Log to inspect acquisition cadence and history. Interpret drift patterns against known failure modes described in the paper: e.g. a lost calibrant signal from a malfunctioning calibrant pump compromised calibration, and an order-of-magnitude sensitivity drop reduced precision, both visible in the tracked m/z and intensity timelines.

## Related tools

- **QC4Metabolomics** (R/Shiny system whose module_TrackCmp detects peaks for user-defined m/z/RT pairs across QC-pool injections; module_Files ingests new mzML files and module_Warner flags threshold breaches) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Language of the QC4Metabolomics package that implements the tracking and ppm-deviation computation over the long-term QC timeline)
- **Shiny** (Web-application front end presenting tracked m/z, RT, intensity and peak-shape timelines for the long-term QC pool)
- **Docker** (Deployment container for QC4Metabolomics and its ProteoWizard-based mzML conversion pipeline)

## Evaluation signals

- Every tracked injection has observed m/z, RT, intensity and peak shape populated for each compound on the user-defined m/z/RT list
- Deviation is reported in ppm relative to the target m/z; entries are flagged exactly when |deviation| > 20 ppm (the paper's 'serious analytical problems' threshold)
- Tracking is performed per instrument (e.g. separately for the Waters Synapt) rather than pooled across instruments
- Short-term RT deviations are expected to be < 0.05 min; deviations ≥ 0.1 min or m/z shifts > 20 ppm warrant investigation
- Email warnings fire (module_Warner) when the tracked parameter exceeds the user-defined threshold and appear in module_Log

## Limitations

- Between-batch RT differences can be at least twice the short-term (< 0.05 min) variation, so a single threshold must tolerate batch-to-batch offsets
- Instruments will typically not alert the user or stop acquisition if calibration is inaccurate — silent calibration drift (e.g. a failing calibrant pump) is only caught by this monitoring, not by the instrument
- Loss of calibration in a limited mass region, nonobvious contaminants, or ion suppression/enhancement affecting only certain peaks are failure modes that require tracking the right compounds to detect
- The 20 ppm flag is a prompt-to-investigate signal, not a fix; root causes (e.g. the June 2022 order-of-magnitude sensitivity drop) must be diagnosed manually

## Evidence

- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs: "module_TrackCmp detects peaks based
on a user-defined list of m/z and
retention time (RT) pairs"
- [methods] module_Warner can send emails warning when a quality parameter exceeds a user defined threshold: "module_Warner can send emails warning
when a quality parameter exceeds a user defined threshold"
- [abstract] extracting diagnostic information such as observed m/z , retention time, intensity and peak shape: "extracting diagnostic information such as observed m/z , retention time, intensity and peak shape"
- [results] RT and m/z deviation data for our long-term QC pool (MetNexs) analyzed within all batches: "RT and m/z deviation data for our long-term QC pool (MetNexs) analyzed within all batches"
- [results] in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
