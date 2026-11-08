---
name: qc-mz-deviation-timeline-monitoring
description: Use when raw vendor MS files (converted to mzML/mzXML/mzData) from pooled-QC-injected
  batches must be screened for m/z or RT deviation, calibrant-signal loss, or peak-shape
  degradation on a specific instrument.
when_to_use_negative:
- Input is already a processed feature table rather than raw/instrument-acquired mzML
  files whose QC must be tracked per injection
- The goal is postanalysis correction of intensity drifts (e.g. QC-MXP-style drift
  correction) rather than detection/monitoring of QC parameter deviations
- The question concerns problems QC4Metabolomics explicitly cannot see, such as loss
  of calibration in a limited mass region, nonobvious contaminants, or ion suppression
  in specific chromatographic regions
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: Core Shiny/R package whose modules (module_Files, module_TrackCmp, module_Contaminants,
    module_Warner) perform file ingestion, m/z-RT tracking, contaminant monitoring
    and threshold alerting for QC time lines
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Serves the web dashboard on which m/z deviation, RT, intensity and peak-shape
    time lines are presented
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Implementation language; provides the package ecosystem used for QC data handling
    and plotting (e.g. with ggplot2 and data.table)
- name: Docker
  role: Hosts the ProteoWizard raw-to-mzML converter and docker-compose deployment
    so the QC pipeline runs OS-agnostic
- name: mass spectrometry (MS)
  role: 'Data source: untargeted MS-based metabolomics acquisitions whose calibration
    and peak quality are monitored'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:f9e29dc2da2611250335c84bf331a50ae33fee2e6a6d7783e5197e6a5e6222ba
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
  - mass spectrometry (MS)
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# qc-mz-deviation-timeline-monitoring

## Summary

Retrospective/real-time QC skill for untargeted MS metabolomics that tracks observed m/z deviation, retention time, intensity and peak shape of user-defined compounds (calibrant ions, pooled-QC analytes, contaminants) over analysis time, exposing instrument-level failures such as calibration offsets and calibrant-pump loss.

## When to use

Use when raw vendor MS files (converted to mzML/mzXML/mzData) from pooled-QC-injected batches must be screened for m/z or RT deviation, calibrant-signal loss, or peak-shape degradation on a specific instrument.

## When NOT to use

- Input is already a processed feature table rather than raw/instrument-acquired mzML files whose QC must be tracked per injection
- The goal is postanalysis correction of intensity drifts (e.g. QC-MXP-style drift correction) rather than detection/monitoring of QC parameter deviations
- The question concerns problems QC4Metabolomics explicitly cannot see, such as loss of calibration in a limited mass region, nonobvious contaminants, or ion suppression in specific chromatographic regions

## Inputs

- Vendor raw MS files (mzML, mzXML, mzData) from untargeted metabolomics batches with pooled QC samples injected every 5-10 injections
- User-defined list of m/z and retention time (RT) pairs for compounds to track (calibrant ions, long-term QC pool compounds such as MetNexs, contaminants)
- List of known contaminant ions (nearly 800 entries used by module_Contaminants)
- User-defined warning thresholds for quality parameters

## Outputs

- Web dashboard (Shiny) time lines of m/z deviation, RT deviation, intensity, and peak shape (tailing factor, asymmetry factor) per instrument
- Flags/warnings (e.g. email from module_Warner) when a quality parameter exceeds its threshold
- Diagnostic interpretation of QC events (e.g. calibration offset in May/June 2022; calibrant-pump failure at end of 2022/beginning of 2023)
- Log of module activities (module_Log) and calendar overview of productivity (module_Productivity)

## How to apply

Set up QC4Metabolomics so vendor raw files are copied from the instrument PC to a network drive and converted to mzML with ProteoWizard in Docker; module_Files then ingests newly acquired files and module_File_info extracts instrument name, project name, ionization mode and sample name. In module_TrackCmp, define a list of m/z and retention time (RT) pairs for calibrant and QC-pool compounds and plot observed m/z deviation versus analysis time (e.g. in R/ggplot2) to expose systematic calibration offsets such as the small offset seen in May/June 2022 on the Waters Synapt. Plot calibrant-compound intensity over time to detect intermittent-then-complete signal loss (end of 2022/beginning of 2023), which pointed to a malfunctioning calibrant pump whose flow was too low for reliable calibration. Track peak shape (tailing factor and asymmetry factor) per injection to flag progressive column degradation, and optionally overlay ~800 known contaminant ions from module_Contaminants to corroborate diagnostics. Configure module_Warner to email alerts when a quality parameter (m/z deviation, RT deviation, intensity, peak shape) exceeds a user-defined threshold, turning retrospective dashboards into near-real-time monitoring.

## Related tools

- **QC4Metabolomics** (Core Shiny/R package whose modules (module_Files, module_TrackCmp, module_Contaminants, module_Warner) perform file ingestion, m/z-RT tracking, contaminant monitoring and threshold alerting for QC time lines) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Serves the web dashboard on which m/z deviation, RT, intensity and peak-shape time lines are presented) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Implementation language; provides the package ecosystem used for QC data handling and plotting (e.g. with ggplot2 and data.table))
- **Docker** (Hosts the ProteoWizard raw-to-mzML converter and docker-compose deployment so the QC pipeline runs OS-agnostic)
- **mass spectrometry (MS)** (Data source: untargeted MS-based metabolomics acquisitions whose calibration and peak quality are monitored)

## Examples

```
Rscript -e "library(QC4Metabolomics); module_TrackCmp(track_cmp = list(mz = c(118.0865), RT = c(1.2)))"
```

## Evaluation signals

- RT deviations for the long-term QC pool are normally < 0.05 min in the shorter term; batch-to-batch differences can be at least twice as large, so flagged deviations should be judged against this baseline
- A real calibration problem appears as a small systematic offset in m/z deviation across many tracked compounds at the same calendar time (analogous to the May/June 2022 Synapt offset), not as random scatter
- Calibrant loss shows as intermittent then complete disappearance of calibrant-compound intensity (as in end-2022/early-2023), subsequently confirmable by a physical cause such as a malfunctioning calibrant pump
- Peak-shape metrics (tailing factor, asymmetry factor) show a monotonic/progressive trend consistent with column degradation rather than abrupt single-file outliers
- Each tracked time line is per-instrument and per-injection; every newly detected mzML file from module_Files appears in the dashboard database (verifiable via module_Log)

## Limitations

- The instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate - monitoring must come from QC software, not the instrument itself
- Certain failure modes are missed by this compound-tracking approach: loss of calibration in a limited mass region, nonobvious contaminants, and ion suppression in specific chromatographic regions or affecting only certain peaks
- RT deviations between different batches can be at least twice as large as within-batch (< 0.05 min) deviations, so thresholds must account for batch effects
- When the calibrant signal is too low (pump malfunction), the data can no longer be used reliably for calibration

## Evidence

- [abstract] extracting diagnostic information such as observed m/z , retention time, intensity and peak shape: "extracting diagnostic information such as observed m/z , retention time, intensity and peak shape"
- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [results] In May/June of 2022 a small systematic offset was observed: "In May/June of 2022 a small systematic offset was observed"
- [results] the pump responsible for the flow of calibrant was malfunctioning and thus the calibration signal was too low to be used reliably for calibration: "the pump responsible for the flow of calibrant was malfunctioning and thus the calibration signal was too low to be used reliably for calibration"
- [methods] module_Contaminants uses a list of nearly 800 known contaminant ions: "module_Contaminants uses a list of nearly 800 known contaminant ions"
- [results] in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
