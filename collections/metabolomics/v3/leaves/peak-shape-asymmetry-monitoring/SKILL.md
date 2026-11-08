---
name: peak-shape-asymmetry-monitoring
description: Use when a long-term LC-MS QC pool is tracked over weeks/months and you
  need to flag progressive column degradation from changing tailing factor or asymmetry
  factor of user-defined tracked compounds.
when_to_use_negative:
- Input is already a corrected, normalized feature table with no per-injection raw
  peak metadata — recompute peak shape from raw mzML-derived chromatograms instead.
- The question is intensity drift correction rather than chromatographic quality —
  QC-MXP-style drift correction is the appropriate tool.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: QC4Metabolomics
  role: R package / Shiny dashboard whose module_TrackCmp detects peaks for user-defined
    m/z and RT pairs and reports peak shape; deployed via Docker
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Analysis language for loading time-line QC data (data.table) and plotting
    peak-shape trends
- name: ggplot2
  role: Plot tailing/asymmetry metrics versus analysis time to expose progressive
    column degradation
- name: Shiny
  role: Web dashboard on which peak-shape results are presented in (near) real time
    or retrospectively
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:f8b1c658e8a0a2b680fe26609ffb7131257ff9173aac6eae9d0525abeb73e5f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - QC4Metabolomics
  - R
  - Shiny
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# peak-shape-asymmetry-monitoring

## Summary

Track chromatographic peak-shape metrics (tailing factor and asymmetry factor) per injection in untargeted LC-MS metabolomics to detect progressive column degradation, as demonstrated retrospectively on a Waters Synapt instrument using the QC4Metabolomics Shiny dashboard.

## When to use

Use when a long-term LC-MS QC pool is tracked over weeks/months and you need to flag progressive column degradation from changing tailing factor or asymmetry factor of user-defined tracked compounds.

## When NOT to use

- Input is already a corrected, normalized feature table with no per-injection raw peak metadata — recompute peak shape from raw mzML-derived chromatograms instead.
- The question is intensity drift correction rather than chromatographic quality — QC-MXP-style drift correction is the appropriate tool.

## Inputs

- mzML/mzXML/mzData files of pooled QC injections registered by module_Files
- Tracked-compound table of user-defined m/z and RT pairs for module_TrackCmp
- Per-injection QC metrics table (observed m/z, RT, intensity, tailing factor, asymmetry factor)

## Outputs

- Peak-shape time-line plots (tailing factor / asymmetry factor vs analysis time) per tracked compound
- PNG panels of each diagnostic time-line
- Summary table of detected QC events with dates and affected parameters

## How to apply

Load retrospective QC time-line data (observed m/z, retention time, intensity, peak shape per injection) from the long-term QC pool into R with data.table. Filter to the instrument of interest (e.g., the Waters Synapt) and the user-defined list of m/z and RT pairs handled by module_TrackCmp, which detects peaks from that list. Plot tailing factor and asymmetry factor versus analysis time with ggplot2; a progressive upward trend in tailing/asymmetry over successive injections indicates column degradation, whereas short-term RT deviations should normally stay below 0.05 min. Optionally overlay contaminant-ion levels from module_Contaminants (a list of nearly 800 known contaminant ions) on the same time line to corroborate chromatographic problems, then save each diagnostic time-line panel as a PNG and assemble a summary table of QC events with dates and affected parameters.

## Related tools

- **QC4Metabolomics** (R package / Shiny dashboard whose module_TrackCmp detects peaks for user-defined m/z and RT pairs and reports peak shape; deployed via Docker) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Analysis language for loading time-line QC data (data.table) and plotting peak-shape trends)
- **ggplot2** (Plot tailing/asymmetry metrics versus analysis time to expose progressive column degradation)
- **Shiny** (Web dashboard on which peak-shape results are presented in (near) real time or retrospectively)

## Evaluation signals

- Each tracked compound has a tailing factor and asymmetry factor computed per injection, plotted against analysis time (not a single batch-level value).
- Time-line is filtered to one instrument (e.g., the Waters Synapt) and the user-defined m/z/RT pair list before plotting.
- A progressive upward drift in tailing/asymmetry is distinguishable from batch-to-batch RT jumps, which can be at least twice the <0.05 min short-term deviation.
- Diagnostic panels are saved as PNG files and accompanied by a summary table listing QC event dates and affected parameters.
- Contaminant-ion time lines from module_Contaminants corroborate the peak-shape interpretation.

## Limitations

- Instruments typically will not alert the user or stop acquisition when quality degrades, so peak-shape problems are only caught if monitoring is in place; retrospectively, the degradation window may already have compromised data quality.
- QC4Metabolomics does not cover nonobvious contaminants, ion suppression in specific chromatographic regions, or loss of calibration limited to a mass region — peak-shape monitoring must be complemented by m/z-deviation and contaminant tracking.
- Batch-to-batch RT differences can be at least twice the short-term (<0.05 min) deviations, so thresholds must tolerate batch effects when judging time trends.

## Evidence

- [abstract] extracting diagnostic information such as observed m/z , retention time, intensity and peak shape: "extracting diagnostic information such as observed m/z , retention time, intensity and peak shape"
- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [methods] module_Contaminants uses a list of nearly 800 known contaminant ions: "module_Contaminants uses a list of nearly 800 known contaminant ions"
- [results] in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [readme] Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/: "Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/"
