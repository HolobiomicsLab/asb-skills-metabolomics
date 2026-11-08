---
name: interactive-dashboard-plotting
description: Use when mzML/mzXML files from untargeted MS-based metabolomics runs
  are being added incrementally (or a whole batch needs retrospective review) and
  extracted diagnostic values — observed m/z, retention time, intensity, peak shape,
  calibrant signal — must be plotted interactively so acquisition.
when_to_use_negative:
- Input is already a processed feature table or a static figure is the only deliverable
  — the value here is live/interactive re-rendering as new mzML files arrive, not
  one-off plotting.
- Targeted batch correction or intensity-drift correction is the goal; QC-MXP handles
  drift correction, while this skill only visualizes the diagnostics that expose the
  drift.
- Automated alerting without human inspection is sufficient — module_Warner sends
  emails on threshold exceedance, so building dashboard views adds little when no
  analyst will look at them.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: QC4Metabolomics
  role: The R package / Shiny app whose dashboard presents the extracted diagnostics;
    source of the plotting code being reused for this skill.
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: R web framework used to render the interactive dashboard views described in
    the abstract and README.
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Provides the docker-compose deployment (with demo data) so the dashboard can
    be brought up reproducibly before plotting.
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Underlying language; QC4Metabolomics leverages the rich ecosystem of R packages
    to build the dashboard plots.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:fd7c5c932c7fef8bf035c8ca82ac4ec672b86184be46a156faca6dcd0881dac2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# interactive-dashboard-plotting

## Summary

Render quality-control diagnostics (QC sample trends, contaminant levels, calibration drift) as live views on an R Shiny web dashboard so analysts can spot instrument problems in (near) real time or retrospectively.

## When to use

Use when mzML/mzXML files from untargeted MS-based metabolomics runs are being added incrementally (or a whole batch needs retrospective review) and extracted diagnostic values — observed m/z, retention time, intensity, peak shape, calibrant signal — must be plotted interactively so acquisition problems are caught before the run finishes.

## When NOT to use

- Input is already a processed feature table or a static figure is the only deliverable — the value here is live/interactive re-rendering as new mzML files arrive, not one-off plotting.
- Targeted batch correction or intensity-drift correction is the goal; QC-MXP handles drift correction, while this skill only visualizes the diagnostics that expose the drift.
- Automated alerting without human inspection is sufficient — module_Warner sends emails on threshold exceedance, so building dashboard views adds little when no analyst will look at them.

## Inputs

- mzML / mzXML / mzData files registered by module_Files (via directory traversal or a text file of file paths)
- Peak-detection results stored in mariaDB: observed m/z, retention time, intensity, peak shape for tracked compounds and ~800 contaminant ions
- Calibration (m/z deviation) and RT deviation data for long-term QC pools such as MetNexs

## Outputs

- Interactive Shiny web dashboard with time-series plots of m/z and RT deviation, compound intensity trends, calibrant signal loss flags
- Three contaminant monitoring views: time view, sample-composition barplot, heatmap clustered over analysis time
- Calendar overview from module_Productivity and activity log from module_Log rendered on the same dashboard

## How to apply

Deploy QC4Metabolomics via the provided Docker images and docker-compose file (demo data included), then let the Shiny web application read extracted diagnostics from the mariaDB database that the R-package modules populate. Build the dashboard views with R/ggplot2-style plotting inside Shiny: time-series plots of m/z and RT deviations (short-term deviations are normally <0.05 min while between-batch differences can be at least twice as large), intensity plots for user-defined tracked compounds such as tryptophan in long-term QC pools (MetNexs), and the three contaminant views from module_Contaminants (level over time, per-sample composition barplot sorted by intensity, heatmap clustered over analysis time) using the bundled list of nearly 800 known contaminant ions. The dashboard is the presentation layer only: module_Warner handles email alerts when a parameter exceeds a user-defined threshold, and module_Log/module_Debug surface the underlying activity log and technical setup info. Validate that plots trace back to database rows keyed by the file paths inserted by module_Files.

## Related tools

- **QC4Metabolomics** (The R package / Shiny app whose dashboard presents the extracted diagnostics; source of the plotting code being reused for this skill.) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (R web framework used to render the interactive dashboard views described in the abstract and README.) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Provides the docker-compose deployment (with demo data) so the dashboard can be brought up reproducibly before plotting.) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Underlying language; QC4Metabolomics leverages the rich ecosystem of R packages to build the dashboard plots.)

## Examples

```
docker compose up -d   # launches the Shiny dashboard (QC4Metabolomics) with demo data; open the web UI to browse RT/m/z deviation plots and the three contaminant views
```

## Evaluation signals

- All three contaminant views (time view, sample-composition barplot, heatmap) render on the Shiny dashboard for the demo data shipped with the docker-compose setup.
- Every plotted point traces back to a row in the mariaDB database whose file path was inserted by module_Files (no orphan data points).
- RT deviation plot reproduces the expected magnitude: short-term deviations <0.05 min, between-batch differences at least twice as large (MetNexs long-term QC pool).
- Known retrospective events from the paper are visible in the plots: the May/June 2022 calibration offset, the June order-of-magnitude sensitivity drop, the intermittent-then-complete calibrant signal loss, and the three-week tryptophan intensity decrease.
- Dashboard re-renders without manual restart when new mzML files are added by module_File_schedule, i.e. the 'real-time' claim is testable by appending a file and waiting for it to appear.

## Limitations

- Instruments typically do not alert the user or stop acquisition if calibration is inaccurate — the dashboard only shows the problem after the diagnostic data exist, so real-time value depends on how quickly new files are copied and converted to mzML.
- Interpretation of drift in a limited mass region, nonobvious contaminants, or ion suppression/enhancement affecting only certain peaks requires domain judgment; the dashboard surfaces the signal but does not diagnose the cause automatically.
- The Shiny dashboard is the presentation layer — if the mariaDB database is empty (e.g. module_Files never found new files) the plots will be blank with no error distinguishing 'no data yet' from 'pipeline broken'.

## Evidence

- [abstract] Real-time monitoring via Shiny dashboard: "We therefore introduce QC4Metabolomics , a real-time quality control monitoring software for untargeted metabolomics data."
- [abstract] What is plotted: [evidence span withheld — anchor status: not_found]
- [abstract] Open-source Shiny app + Docker: "The Shiny app is available as open-source code at https://github.com/stanstrup/QC4Metabolomics. Docker images and a docker-compose setup file are also provided for easy deployment, along with demo"
- [intro] Motivation for real-time viewing: "would enable users to spot and correct for analytical problems during analysis, thus ensuring the acquisition of the highest quality data."
- [results] Threshold expectation for RT plots: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [results] Contaminant views available: [evidence span withheld — anchor status: not_found]
- [methods] Contaminant ion list size: "module_Contaminants uses a list of nearly 800 known contaminant ions"
- [methods] Data feeding the dashboard: "module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths and then adds them to a table in the"
- [methods] Email alerting is separate from dashboard: "module_Warner can send emails warning when a quality parameter exceeds a user defined threshold"
- [results] Long-term QC pool name: "RT and m/z deviation data for our long-term QC pool (MetNexs) analyzed within all batches"
- [results] Calibrant failure visible in plots: "the calibrant signal was lost intermittently, then completely, compromising calibration"
- [intro] Dashboard is for data quality documentation: "documents data quality postanalysis using an intuitive dashboard"
