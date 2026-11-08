---
name: metabolomics-quality-control-monitoring
description: Use when untargeted MS-based metabolomics files (mzML, mzXML, mzData)
  are being acquired or already acquired and you need to monitor QC parameters such
  as m/z deviation, retention time deviation, intensity drift, calibration offset,
  or contaminant levels in (near) real time or retrospectively.
when_to_use_negative:
- Input is already a processed feature table or corrected intensity matrix — QC4Metabolomics
  expects vendor raw or mzML-format untargeted MS files acquired during or after an
  LC-MS run, not ready-made downstream results.
- You need targeted quantitation of a small panel of analytes with a validated calibration
  curve — this skill is for untargeted metabolomics QC monitoring, not targeted assay
  validation.
- Data are not mass-spectrometry-based (e.g. NMR spectra, imaging data) or lack the
  pooled-QC injection cadence needed to anchor intensity-drift and calibration monitoring.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: QC4Metabolomics
  role: Core Shiny web application and R package that performs real-time and retrospective
    QC monitoring by tracking user-defined compounds and contaminants in untargeted
    MS-based metabolomics data.
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Web framework used to render the QC dashboard for real-time and retrospective
    inspection of m/z, RT, intensity, and contaminant views.
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Container platform providing the docker-compose deployment of QC4Metabolomics,
    the OS-agnostic ProteoWizard raw-to-mzML converter, and demo data for setup validation.
- name: QCScreen
  role: Prior software referenced in the introduction that gives an overview of quality
    and stability parameters; QC4Metabolomics complements it with real-time and dashboard-based
    monitoring.
- name: QC-MXP
  role: Prior software providing tools for correcting intensity drifts; QC4Metabolomics
    also enables postanalysis corrections of intensity drifts anchored on pooled QC
    samples.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:dab0b8b05016f7e3c828154054c55571b1eb966dd87abed3c249d5c283f628d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC-MXP
  - QC4Metabolomics
  - QCScreen
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# metabolomics-quality-control-monitoring

## Summary

Real-time and retrospective quality control of untargeted mass spectrometry-based metabolomics data using QC4Metabolomics, a Shiny web application and R package that tracks user-defined compounds and contaminants across acquired mzML files.

## When to use

Use when untargeted MS-based metabolomics files (mzML, mzXML, mzData) are being acquired or already acquired and you need to monitor QC parameters such as m/z deviation, retention time deviation, intensity drift, calibration offset, or contaminant levels in (near) real time or retrospectively.

## When NOT to use

- Input is already a processed feature table or corrected intensity matrix — QC4Metabolomics expects vendor raw or mzML-format untargeted MS files acquired during or after an LC-MS run, not ready-made downstream results.
- You need targeted quantitation of a small panel of analytes with a validated calibration curve — this skill is for untargeted metabolomics QC monitoring, not targeted assay validation.
- Data are not mass-spectrometry-based (e.g. NMR spectra, imaging data) or lack the pooled-QC injection cadence needed to anchor intensity-drift and calibration monitoring.

## Inputs

- Vendor raw mass spectrometry files copied to a network drive (Windows batch script)
- Converted mzML / mzXML / mzData untargeted metabolomics files
- User-defined list of m/z and retention time (RT) pairs for compounds to track
- List of contaminant ions (bundled list of nearly 800 known contaminant ions, or a user-supplied list)
- Pooled QC samples injected at regular intervals (every 5–10 injections) among biological samples
- Instrument and sample metadata available from the raw files

## Outputs

- Web dashboard (Shiny) of QC parameters per file: observed m/z, retention time, intensity, and peak shape
- Contaminant monitoring views: contaminant level over time, per-sample contaminant composition barplot, and heat map clustering contaminants over analysis time
- Calendar overview of instrument productivity (module_Productivity)
- Log of module activities (module_Log) and technical setup information (module_Debug)
- Email warnings when a quality parameter exceeds a user-defined threshold (module_Warner)
- Postanalysis corrections of intensity drifts (e.g. via QC-MXP)

## How to apply

Deploy QC4Metabolomics via the provided docker-compose setup (Docker images plus demo data) and open the Shiny web application. Copy vendor raw files from the instrument PC to an accessible network drive using a Windows batch script, convert them to mzML with ProteoWizard inside Docker, and let module_Files detect new files by traversing a directory or reading from a text file containing file paths, adding them to a mariaDB table. module_File_schedule then queues detection, module_File_info extracts sample metadata (instrument name, project name, ionization mode, sample name), and module_TrackCmp detects peaks for user-defined m/z and retention time pairs to extract diagnostic information (observed m/z, retention time, intensity, peak shape). Enable module_Contaminants with a list of nearly 800 known contaminant ions (or a user-supplied list) to monitor contaminant levels across time, per-sample composition, and heat-map views, and configure module_Warner to send email alerts when a quality parameter exceeds a user-defined threshold. Interpret the results on the intuitive dashboard, noting that RT deviations are normally less than 0.05 min in the shorter term while differences between batches can be at least twice as large, and use pooled QC samples injected every 5–10 injections to anchor intensity-drift corrections.

## Related tools

- **QC4Metabolomics** (Core Shiny web application and R package that performs real-time and retrospective QC monitoring by tracking user-defined compounds and contaminants in untargeted MS-based metabolomics data.) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Web framework used to render the QC dashboard for real-time and retrospective inspection of m/z, RT, intensity, and contaminant views.) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Container platform providing the docker-compose deployment of QC4Metabolomics, the OS-agnostic ProteoWizard raw-to-mzML converter, and demo data for setup validation.)
- **QCScreen** (Prior software referenced in the introduction that gives an overview of quality and stability parameters; QC4Metabolomics complements it with real-time and dashboard-based monitoring.)
- **QC-MXP** (Prior software providing tools for correcting intensity drifts; QC4Metabolomics also enables postanalysis corrections of intensity drifts anchored on pooled QC samples.)

## Examples

```
docker-compose up -d  # deploy QC4Metabolomics; then open the Shiny app and configure module_TrackCmp with user-defined m/z/RT pairs and module_Contaminants with the bundled ~800-ion list
```

## Evaluation signals

- Verify that all three contaminant monitoring views render on the Shiny dashboard: contaminant level over time, per-sample composition barplot sorted by intensity, and a heat map clustering contaminants over analysis time.
- Confirm that diagnostic information for each tracked compound includes observed m/z, retention time, intensity, and peak shape stored in the mariaDB database.
- Check that RT deviations of tracked compounds in the shorter term are normally less than 0.05 min, and that deviations between different batches can be at least twice as large — deviations outside these ranges should be flagged for inspection.
- Validate that contaminant detections are traceable to entries in the bundled list of nearly 800 known contaminant ions (or to the user-supplied list).
- Confirm that module_Warner sends email alerts when a user-defined quality-parameter threshold is exceeded, and that module_Log records the activities of the other modules.

## Limitations

- The instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate; a small systematic offset observed in May/June 2022 and a lost calibrant signal at the end of 2022/beginning of 2023 (caused by a malfunctioning calibrant pump) were only visible through retrospective monitoring.
- QC4Metabolomics monitors files as they are acquired or retrospectively but does not itself stop acquisition; correcting intensity drifts requires postanalysis steps (e.g. QC-MXP) anchored on pooled QC samples injected every 5–10 injections.
- Sensitivity can decrease by an order of magnitude (observed in June 2022), reducing precision; such changes must be detected through the dashboard rather than by the instrument itself.
- Some quality problems (loss of calibration in a limited mass region, nonobvious contaminants, ion suppression in specific chromatographic regions) may be nonobvious and require the combined m/z, RT, intensity, and peak-shape diagnostics to be noticed.

## Evidence

- [abstract] QC4Metabolomics monitors files as they are acquired or retrospectively by tracking any user-defined compound(s): "QC4Metabolomics monitors files as they are acquired or retrospectively by tracking any user-defined compound(s)"
- [abstract] extracting diagnostic information such as observed m/z , retention time, intensity and peak shape: "extracting diagnostic information such as observed m/z , retention time, intensity and peak shape"
- [methods] module_Contaminants uses a list of nearly 800 known contaminant ions: "module_Contaminants uses
a list of
nearly 800 known contaminant ions"
- [results] in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [results] The instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate: "The instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate"
- [intro] The QC sample is injected at regular intervals, i.e. every 5–10 injections among the biological samples.: "The QC sample is injected at regular intervals, i.e. every 5–10 injections among the biological samples."
- [abstract] Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data.: "Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data."
- [methods] module_Files looks for newly acquired files (mzML, mzXML, mzData etc.): "module_Files looks for newly acquired
files (mzML, mzXML, mzData etc.)"
