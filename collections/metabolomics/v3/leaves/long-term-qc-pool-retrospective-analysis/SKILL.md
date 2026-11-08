---
name: long-term-qc-pool-retrospective-analysis
description: Use when untargeted MS metabolomics data (mzML from pooled QC injections)
  must be retro-inspected across batches for systematic m/z offsets, calibrant intensity
  loss, or tailing/asymmetry degradation on a specific instrument such as a Waters
  Synapt.
when_to_use_negative:
- You need postanalysis correction of intensity drifts rather than diagnostics — use
  drift-correction tools such as QC-MXP instead
- The goal is real-time alerting during acquisition with threshold-triggered email
  warnings — that is module_Warner's role, not retrospective reconstruction
- Input is already a processed feature table without per-injection QC metrics (m/z,
  RT, intensity, peak shape) — module_TrackCmp requires raw-derived tracked-compound
  peaks
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: Real-time and retrospective QC dashboard that tracks user-defined compounds
    and stores per-injection m/z, RT, intensity, and peak-shape metrics used for the
    retrospective diagnostics
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Analysis environment for loading retrospective QC data (data.table) and rendering
    diagnostic time-line panels (ggplot2)
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Web-application framework presenting the QC dashboards from which retrospective
    instrument problems are revealed
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Deployment and vendor-raw-to-mzML conversion (ProteoWizard) feeding the QC
    database with OS-agnostic processing
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:57af3e0999419efb29a81251e75a28ec288190dcb5b97cc1366c1ae0e2b9c8a3
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# long-term-qc-pool-retrospective-analysis

## Summary

Retrospective diagnostic analysis of long-term pooled QC (e.g., the MetNexs QC pool) using QC4Metabolomics dashboards to reconstruct instrument-level failure timelines — calibration drift, calibrant signal loss, and column-degradation peak-shape changes — that real-time monitoring could have caught during acquisition.

## When to use

Use when untargeted MS metabolomics data (mzML from pooled QC injections) must be retro-inspected across batches for systematic m/z offsets, calibrant intensity loss, or tailing/asymmetry degradation on a specific instrument such as a Waters Synapt.

## When NOT to use

- You need postanalysis correction of intensity drifts rather than diagnostics — use drift-correction tools such as QC-MXP instead
- The goal is real-time alerting during acquisition with threshold-triggered email warnings — that is module_Warner's role, not retrospective reconstruction
- Input is already a processed feature table without per-injection QC metrics (m/z, RT, intensity, peak shape) — module_TrackCmp requires raw-derived tracked-compound peaks

## Inputs

- Long-term pooled QC sample LC-MS runs (vendor raw files copied to network storage and converted to mzML via ProteoWizard in Docker)
- User-defined list of m/z and retention time (RT) pairs for tracked compounds (including calibrant compounds)
- QC4Metabolomics database of per-injection QC metrics (observed m/z, RT deviation, intensity, tailing/asymmetry peak-shape metrics), e.g., the MetNexs long-term QC pool

## Outputs

- Time-line panels of m/z deviation vs. analysis time per instrument (PNG), exposing systematic calibration offsets
- Time-line panels of calibrant-compound intensity/deviation showing intermittent-then-complete signal loss
- Peak-shape (tailing factor and asymmetry factor) time-series showing progressive column degradation
- Summary table of detected QC events with dates and affected parameters

## How to apply

Load the long-term QC pool retrospective data (per-injection observed m/z, retention time, intensity, and peak-shape metrics) into R (data.table/ggplot2) from the QC4Metabolomics dashboard pipeline. Filter the time-line data for the instrument of interest using the user-defined list of m/z and RT pairs handled by module_TrackCmp, then plot m/z deviation versus analysis time to expose systematic calibration offsets (e.g., the small offset seen in May/June 2022 on the Waters Synapt, accompanied by an order-of-magnitude sensitivity drop in June that reduced precision). Plot tracked calibrant-compound intensity over time to reveal intermittent-then-complete signal loss (end-2022/beginning-2023, later confirmed as a malfunctioning calibrant pump). Plot tailing factor and asymmetry factor over time to demonstrate progressive column degradation, optionally overlaying contaminant-ion levels from module_Contaminants (~800 known contaminant ions) to corroborate the interpretation. Instruments will typically not alert the user or stop acquisition when calibration is inaccurate, so these retrospective panels are the primary diagnostic.

## Related tools

- **QC4Metabolomics** (Real-time and retrospective QC dashboard that tracks user-defined compounds and stores per-injection m/z, RT, intensity, and peak-shape metrics used for the retrospective diagnostics) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Analysis environment for loading retrospective QC data (data.table) and rendering diagnostic time-line panels (ggplot2)) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Web-application framework presenting the QC dashboards from which retrospective instrument problems are revealed) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Deployment and vendor-raw-to-mzML conversion (ProteoWizard) feeding the QC database with OS-agnostic processing) — https://github.com/stanstrup/QC4Metabolomics

## Examples

```
Rscript -e 'library(data.table); qc <- fread("metnexs_qc_pool.tsv"); synapt <- qc[instrument=="Waters Synapt"]; ggplot(synapt, aes(acq_time, mz_dev)) + geom_line() + ggsave("mz_offset_timeline.png")'
```

## Evaluation signals

- m/z deviation plots show a small systematic offset confined to the expected time window (May/June 2022) rather than random scatter
- Calibrant-compound intensity time series shows intermittent dips followed by complete loss (end-2022/beginning-2023), consistent with the confirmed malfunctioning calibrant pump
- Tailing factor and asymmetry factor trends show monotonic degradation rather than batch-level jumps
- RT deviations remain within the expected regime: normally less than 0.05 min short-term, with between-batch differences up to twice as large used to bound normal variation
- Summary table entries carry dates and affected parameters matching the plotted event windows, and contaminant overlay corroborates (rather than contradicts) the instrument-level interpretation

## Limitations

- Instruments typically do not alert or stop acquisition when calibration is inaccurate, so calibrant-related failures are only detected by explicit tracking of calibrant signals
- Some failure modes are invisible to tracked-compound monitoring alone, e.g., loss of calibration in a limited mass region, nonobvious contaminants, and ion suppression in specific chromatographic regions or affecting only certain peaks
- RT deviations between batches can be at least twice the short-term (<0.05 min) variation, so batch-to-batch shifts must not be misread as instrument faults
- The workflow depends on raw files being copied from the instrument PC to network storage and converted to mzML; gaps in that pipeline create gaps in the retrospective timeline

## Evidence

- [abstract] We report herein real-world examples where QC4Metabolomics easily reveals analytical problems retrospectively that could have been immediately addressed with real-time monitoring: "We report herein real-world examples where QC4Metabolomics easily reveals analytical problems retrospectively that could have been immediately addressed with real-time monitoring"
- [results] The pump responsible for the flow of calibrant was malfunctioning and thus the calibration signal was too low to be used reliably for calibration: [evidence span withheld — anchor status: not_found]
- [results] The instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate: "The instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate"
- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [results] in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [readme] Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/: "Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/"
