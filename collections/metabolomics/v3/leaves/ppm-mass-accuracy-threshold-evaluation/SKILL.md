---
name: ppm-mass-accuracy-threshold-evaluation
description: Use when tracking known compounds in untargeted LC–MS metabolomics mzML
  files (e.g. Waters Synapt) and you need to detect mass-accuracy drift or calibration
  failure from long-term QC pool data.
when_to_use_negative:
- Input is already a processed feature table or peak-picked data rather than raw vendor
  files / mzML that can be peak-detected against m/z–RT pairs
- The goal is intensity-drift correction rather than mass-accuracy monitoring (QC-MXP
  handles intensity drifts)
- Calibration is known to be accurate and there is no need to monitor m/z stability;
  the skill adds no value if deviation checks are redundant
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: R/Shiny framework that implements module_TrackCmp for peak detection on m/z–RT
    pairs and module_Warner for threshold-based email warnings
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Language in which module_TrackCmp and the deviation computation are implemented
    via QC4Metabolomics's package ecosystem
- name: Shiny
  role: Web dashboard used to display the tracked compound quality parameters and
    deviation time-lines
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Deployment container for QC4Metabolomics (including the ProteoWizard mzML
    converter) used when converting vendor raw data
- name: module_TrackCmp
  role: Detects peaks based on a user-defined list of m/z and RT pairs and extracts
    observed m/z, RT, intensity and peak shape diagnostics per injection
- name: module_Warner
  role: Sends emails when a quality parameter (here the m/z ppm deviation) exceeds
    the user-defined 20 ppm threshold
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:fad4eb467ed8230dd695653fd0e5809c4241d37badece8582f7015a634225ec5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  - module_TrackCmp
  - module_Warner
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# ppm-mass-accuracy-threshold-evaluation

## Summary

A QC skill that computes per-injection m/z deviations (in ppm) from a user-defined list of target m/z–retention time pairs and flags deviations exceeding a 20 ppm threshold, indicating serious analytical problems that should be addressed promptly.

## When to use

Use when tracking known compounds in untargeted LC–MS metabolomics mzML files (e.g. Waters Synapt) and you need to detect mass-accuracy drift or calibration failure from long-term QC pool data.

## When NOT to use

- Input is already a processed feature table or peak-picked data rather than raw vendor files / mzML that can be peak-detected against m/z–RT pairs
- The goal is intensity-drift correction rather than mass-accuracy monitoring (QC-MXP handles intensity drifts)
- Calibration is known to be accurate and there is no need to monitor m/z stability; the skill adds no value if deviation checks are redundant

## Inputs

- mzML (mzXML, mzData) files from long-term QC pool injections (e.g. MetNexs) acquired on the Waters Synapt instrument
- User-defined list of m/z and retention time (RT) pairs for compounds to track

## Outputs

- Per-injection m/z deviation time-line (in ppm) for each tracked compound, with entries exceeding the 20 ppm threshold flagged
- Table file containing the deviation time-line with flagged entries
- Email warnings via module_Warner when the quality parameter exceeds the threshold

## How to apply

Load the MetNexs long-term QC pool mzML files acquired on the Waters Synapt instrument into QC4Metabolomics. Run module_TrackCmp on a user-defined list of m/z and retention time (RT) pairs to detect peaks and extract observed m/z, retention time, intensity and peak shape diagnostics for each injection. Compute m/z deviations in ppm from the target m/z values for each tracked compound across the time-line, then flag injections where the deviation exceeds 20 ppm — the threshold the article reports indicates serious analytical problems. Wire the results into module_Warner so emails are sent when the quality parameter exceeds the user-defined threshold, and save the per-injection deviation time-line with flagged entries as a table file. The rationale is that instruments typically do not alert the user or stop acquisition when calibration is inaccurate, so this retrospective/real-time check is the safety net for calibration loss (e.g. the reported calibrant-signal loss where the pump was malfunctioning).

## Related tools

- **QC4Metabolomics** (R/Shiny framework that implements module_TrackCmp for peak detection on m/z–RT pairs and module_Warner for threshold-based email warnings) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Language in which module_TrackCmp and the deviation computation are implemented via QC4Metabolomics's package ecosystem)
- **Shiny** (Web dashboard used to display the tracked compound quality parameters and deviation time-lines) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Deployment container for QC4Metabolomics (including the ProteoWizard mzML converter) used when converting vendor raw data)
- **module_TrackCmp** (Detects peaks based on a user-defined list of m/z and RT pairs and extracts observed m/z, RT, intensity and peak shape diagnostics per injection)
- **module_Warner** (Sends emails when a quality parameter (here the m/z ppm deviation) exceeds the user-defined 20 ppm threshold)

## Examples

```
Rscript -e 'QC4Metabolomics::module_TrackCmp(mzml_dir="QC_pool_mzML/", targets=cmp_list); QC4Metabolomics::module_Warner(threshold_ppm=20)'
```

## Evaluation signals

- Observed m/z values extracted by module_TrackCmp match the target m/z within reasonable bounds for well-calibrated injections (deviations well below 20 ppm)
- All injections whose computed m/z deviation exceeds 20 ppm are flagged in the output table
- Per-injection deviation time-line is chronological and covers every file processed from the QC pool
- Email warnings fire (or would fire) for any injection exceeding the 20 ppm threshold, as configured in module_Warner
- Output is a table file per injection/compound pair with observed m/z, RT, intensity, peak shape, and computed ppm deviation columns

## Limitations

- Threshold of 20 ppm is calibrated from the authors' MetNexs long-term QC pool on a Waters Synapt instrument; deviations of that size 'indicate serious analytical problems that should be addressed promptly' — but the absolute number may need re-tuning for other instruments or pools
- Requires vendor raw files be copied to a network drive and converted to mzML before monitoring; raw proprietary formats cannot be processed directly
- m/z deviation alone does not capture RT drift, intensity loss, or ion suppression; other QC parameters must be monitored separately
- Calibration loss can be limited to a specific mass region and the instruments typically do not alert the user or stop acquisition, so the flagged compounds must adequately cover the mass range of interest

## Evidence

- [results] Tracking long-term QC pool deviation threshold: [evidence span withheld — anchor status: not_found]
- [methods] Threshold 20 ppm from Warner module: "module_Warner can send emails warning when a quality parameter exceeds a user defined threshold"
- [methods] module_TrackCmp operation: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [abstract] Extracted diagnostic information: "extracting diagnostic information such as observed m/z , retention time, intensity and peak shape"
- [results] Instruments do not alert on bad calibration: "The instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate"
- [abstract] Docker/Shiny deployment: "The Shiny app is available as open-source code at https://github.com/stanstrup/QC4Metabolomics. Docker images and a docker-compose setup file are also provided for easy deployment"
