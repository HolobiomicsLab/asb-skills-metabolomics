---
name: calibrant-signal-loss-detection
description: Use when a Waters Synapt (or similar untargeted MS metabolomics) run
  shows suspicious m/z deviations or poor mass accuracy and you need to check whether
  the calibrant signal drifted, dropped intermittently, or disappeared entirely.
when_to_use_negative:
- When the input is already a post-correction feature table with no per-injection
  calibrant intensity or m/z-deviation traces — the method requires time-resolved
  QC-pool metrics
- When the goal is correcting intensity drifts postanalysis rather than diagnosing
  instrument-level calibration problems (QC-MXP-style drift correction is a different
  skill)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: Source of the module_TrackCmp peak-detection and dashboard framework used
    to track calibrant m/z/RT pairs and extract per-injection deviations
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Web dashboard used to visualize the calibrant-signal timelines retrospectively
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: R/data.table/ggplot2 workflow used to filter and plot the calibrant-intensity
    and m/z-deviation timelines
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:a17f6f97ae607cd410c623885fa8ae97fd7ccd092d69c0ab9cfb950fe21beeaf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - QC4Metabolomics
  - R
  - Shiny
  - ggplot2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# calibrant-signal-loss-detection

## Summary

Retrospective diagnostic technique from QC4Metabolomics that plots calibrant-compound m/z deviation and intensity over analysis time to detect calibration offsets and calibrant-signal loss (e.g., the May/June 2022 offset and the end-2022/early-2023 intermittent-then-complete calibrant loss caused by a malfunctioning calibrant pump on a Waters Synapt).

## When to use

Use when a Waters Synapt (or similar untargeted MS metabolomics) run shows suspicious m/z deviations or poor mass accuracy and you need to check whether the calibrant signal drifted, dropped intermittently, or disappeared entirely.

## When NOT to use

- When the input is already a post-correction feature table with no per-injection calibrant intensity or m/z-deviation traces — the method requires time-resolved QC-pool metrics
- When the goal is correcting intensity drifts postanalysis rather than diagnosing instrument-level calibration problems (QC-MXP-style drift correction is a different skill)

## Inputs

- Long-term QC pool retrospective QC data (e.g., MetNexs) with per-injection m/z deviation, RT deviation, intensity, and peak-shape metrics
- User-defined list of calibrant-compound m/z and RT pairs
- Vendor raw MS data (mzML/mzXML/mzData) processed through the QC4Metabolomics pipeline into its database

## Outputs

- Time-line panels (PNG) of calibrant m/z deviation versus analysis time
- Time-line panels (PNG) of calibrant-compound intensity over the diagnostic window
- Summary table of detected QC events with dates and affected parameters (e.g., calibration offset May/June 2022; calibrant signal loss end-2022/early-2023)

## How to apply

Load the long-term QC pool (MetNexs) retrospective QC data — per-injection m/z deviation, RT deviation, intensity, and peak-shape metrics — into R with data.table. Using module_TrackCmp-style logic, filter the timeline for the instrument of interest via a user-defined list of m/z and RT pairs for the calibrant compounds, then plot m/z deviation versus analysis time with ggplot2 to expose systematic calibration offsets (e.g., the small systematic offset seen in May/June 2022). Next, plot calibrant-compound intensity over time across the suspect window: an intermittent-then-complete loss of the calibrant signal, with sensitivity dropping by an order of magnitude, indicates the calibrant pump is failing or malfunctioning. Optionally overlay ~800 known contaminant-ion levels from module_Contaminants on the same timeline to corroborate that observed changes are instrument- rather than sample-driven. Save each diagnostic time-line panel as a PNG and compile a dated summary table of affected QC parameters.

## Related tools

- **QC4Metabolomics** (Source of the module_TrackCmp peak-detection and dashboard framework used to track calibrant m/z/RT pairs and extract per-injection deviations) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Web dashboard used to visualize the calibrant-signal timelines retrospectively) — https://github.com/stanstrup/QC4Metabolomics
- **R** (R/data.table/ggplot2 workflow used to filter and plot the calibrant-intensity and m/z-deviation timelines)

## Examples

```
library(data.table); library(ggplot2); qc <- fread('metnexs_qc.csv')[instrument=='Waters Synapt']; ggplot(qc[mz_dev!=NA], aes(x=inj_time, y=mz_dev)) + geom_point() + ggsave('calibration_offset.png')
```

## Evaluation signals

- The m/z-deviation plot shows the small systematic offset reported for May/June 2022 on the Waters Synapt
- The calibrant-intensity timeline reproduces the expected pattern: intermittent loss at end-2022 progressing to complete loss in early 2023
- The loss coincides with a reported order-of-magnitude sensitivity decrease that reduced precision
- Every PNG panel has a time axis spanning the suspect window and is derived from the MetNexs QC-pool data for the specific instrument
- The summary table lists each QC event with a date and the affected parameter (m/z deviation, intensity, RT, peak shape)

## Limitations

- Instruments typically do not alert the user or stop acquisition when calibration is inaccurate, so this diagnostic is retrospective unless real-time monitoring is enabled
- A low calibrant signal means the calibration signal is too low to be used reliably for calibration — quality of subsequent mass calibration on affected injections is already compromised
- The diagnostic relies on availability of long-term QC pool data; without regular pooled-QC injections (every 5–10 injections) no timeline can be reconstructed

## Evidence

- [results] In May/June of 2022 a small systematic offset was observed: "In May/June of 2022 a small systematic offset was observed"
- [results] calibrant signal lost intermittently then completely: "the calibrant signal was lost intermittently, then completely, compromising calibration"
- [results] pump malfunction causing low calibration signal: "the pump responsible for the flow of calibrant was malfunctioning and thus the calibration signal was too low to be used reliably for calibration"
- [methods] module_TrackCmp user-defined m/z RT pairs: "module_TrackCmp detects peaks based
on a user-defined list of m/z and
retention time (RT) pairs"
- [results] instruments do not alert on bad calibration: "The instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate"
- [results] order-of-magnitude sensitivity loss in June: "in June, sensitivity decreased by an order of magnitude which in turn reduced precision"
