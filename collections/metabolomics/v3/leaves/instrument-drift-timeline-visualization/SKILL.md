---
name: instrument-drift-timeline-visualization
description: Use when long-term pooled-QC injections (e.g., MetNexs long-term QC pool)
  tracked per instrument show suspected calibration drift, intermittent calibrant-signal
  loss, or degrading peak shape, and the goal is to plot deviations versus analysis
  time to date and diagnose the malfunction.
when_to_use_negative:
- Input is a postanalysis-corrected feature table or drift-corrected intensities (e.g.,
  from QC-MXP-style drift correction) — the timeline visualization must operate on
  raw per-injection QC metrics, not corrected outputs
- The question is targeted metabolite quantification or statistical batch correction
  rather than diagnosing instrument-level hardware/calibration faults over acquisition
  time
- Only a single batch is available with no pooled-QC injections at regular intervals
  (every 5–10 injections), since timeline trends across batches cannot be established
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: Shiny web application and R package whose dashboard and module_TrackCmp/module_Contaminants
    modules supply and render the per-instrument QC time lines
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Web framework presenting the QC time lines and diagnostic dashboards
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Language of the QC4Metabolomics package; data.table and ggplot2 are used within
    it for timeline filtering and plotting
- name: Docker
  role: Docker images and docker-compose setup provide deployment of QC4Metabolomics
    with demo data for reproducing the retrospective dashboards
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:3679577642cf54ddb3c6f5bf00f21b27c258cf88ceb0052f91281f194f796496
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# instrument-drift-timeline-visualization

## Summary

Visualize instrument-level quality metrics (m/z deviation, RT deviation, calibrant intensity, and peak-shape metrics such as tailing/asymmetry factor) as time lines across batches to retrospectively expose calibration offsets, calibrant-signal loss, and progressive column degradation on instruments like the Waters Synapt, using the QC4Metabolomics Shiny dashboard.

## When to use

Use when long-term pooled-QC injections (e.g., MetNexs long-term QC pool) tracked per instrument show suspected calibration drift, intermittent calibrant-signal loss, or degrading peak shape, and the goal is to plot deviations versus analysis time to date and diagnose the malfunction.

## When NOT to use

- Input is a postanalysis-corrected feature table or drift-corrected intensities (e.g., from QC-MXP-style drift correction) — the timeline visualization must operate on raw per-injection QC metrics, not corrected outputs
- The question is targeted metabolite quantification or statistical batch correction rather than diagnosing instrument-level hardware/calibration faults over acquisition time
- Only a single batch is available with no pooled-QC injections at regular intervals (every 5–10 injections), since timeline trends across batches cannot be established

## Inputs

- Per-injection QC metric time lines (observed m/z deviation, RT deviation, intensity, tailing factor, asymmetry factor) from a long-term QC pool such as MetNexs, loaded via QC4Metabolomics
- User-defined list of m/z and RT pairs for tracked compounds (module_TrackCmp input)
- Converted mzML/mzXML/mzData files registered by module_Files for the instrument under inspection

## Outputs

- ggplot2 time-line panels per diagnostic (m/z deviation vs. analysis time, calibrant intensity decay, peak-shape degradation) exported as PNG files
- Summary table of detected QC events with dates (e.g., May/June 2022 calibration offset; end-2022/early-2023 calibrant loss) and affected parameters

## How to apply

Load per-injection QC metrics (observed m/z, RT, intensity, peak shape) for the long-term QC pool into R with data.table, filter the timeline to the instrument of interest (e.g., a Waters Synapt), and use module_TrackCmp's user-defined list of m/z/RT pairs to build ggplot2 time-line plots of m/z deviation versus analysis time — a small systematic offset confined to May/June 2022 indicates a calibration problem the instrument itself will not flag. Separately plot the calibrant-compound intensity over time: intermittent then complete signal loss at the end of 2022/beginning of 2023 points to a hardware fault, which in the reported case was a malfunctioning calibrant pump. Plot tailing-factor and asymmetry-factor timelines for tracked compounds to detect progressive column degradation, and optionally overlay contaminant-ion levels from module_Contaminants (nearly 800 known contaminant ions) on the same time lines to corroborate the diagnosis. Interpret timelines knowing the paper's baseline: short-term RT deviations are normally <0.05 min while batch-to-batch differences can be twice as large, so judge deviations relative to that scale rather than assuming any shift is an instrument fault.

## Related tools

- **QC4Metabolomics** (Shiny web application and R package whose dashboard and module_TrackCmp/module_Contaminants modules supply and render the per-instrument QC time lines) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Web framework presenting the QC time lines and diagnostic dashboards) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Language of the QC4Metabolomics package; data.table and ggplot2 are used within it for timeline filtering and plotting)
- **Docker** (Docker images and docker-compose setup provide deployment of QC4Metabolomics with demo data for reproducing the retrospective dashboards) — https://github.com/stanstrup/QC4Metabolomics

## Examples

```
Rscript -e 'library(data.table); dt<-fread("metnexs_qc_timeline.csv"); library(ggplot2); ggplot(dt[instrument=="Synapt"],aes(x=acq_time,y=mz_dev))+geom_point()+geom_line(); ggsave("mz_offset_timeline.png")'
```

## Evaluation signals

- The May/June 2022 calibration-offset panel shows a small systematic m/z deviation shift bounded to that date range, not a diffuse spread across all batches
- The calibrant-intensity timeline reproduces the reported pattern: intermittent loss at end of 2022, then complete loss into early 2023, matching the later-confirmed malfunctioning calibrant pump
- RT deviation magnitudes are interpreted against the paper's baseline (<0.05 min short-term; ≥2x between batches), so flagged deviations exceed that scale
- Peak-shape timelines (tailing factor, asymmetry factor) show a monotonic/progressive trend consistent with column degradation rather than random injection noise
- Each diagnostic panel is saved as a PNG and accompanied by a summary table listing event dates and affected parameters

## Limitations

- Instruments typically do not alert the user or stop acquisition when calibration is inaccurate, so timelines only help if QC metrics are actually being tracked during acquisition or retrospectively afterwards
- The skill cannot detect issues invisible to tracked compounds and contaminant lists, such as loss of calibration in a limited mass region, nonobvious contaminants, or ion suppression affecting only certain peaks or specific chromatographic regions
- RT deviations between batches can be at least twice the short-term (<0.05 min) deviations, so cross-batch shifts can masquerade as instrument faults without batch-aware interpretation

## Evidence

- [results] In May/June of 2022 a small systematic offset was observed: "In May/June of 2022 a small systematic offset was observed"
- [results] Calibrant signal loss and pump failure: "the calibrant signal was lost intermittently, then completely, compromising calibration"
- [results] Pump malfunction root cause: "the pump responsible for the flow of calibrant was malfunctioning and thus the calibration signal was too low to be used reliably for calibration"
- [results] RT deviation baseline scale: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [methods] module_TrackCmp input definition: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [methods] module_Contaminants scope: "module_Contaminants uses a list of nearly 800 known contaminant ions"
- [results] Instruments do not guard against bad calibration: "The instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate"
- [results] MetNexs long-term QC pool dataset: "RT and m/z deviation data for our long-term QC pool (MetNexs) analyzed within all batches"
