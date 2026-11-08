---
name: qc-compound-peak-detection
description: Use when untargeted MS-based metabolomics mzML files (including pooled
  QC injections) must be monitored for a defined compound list — e.g. tracking the
  MetNexs long-term QC pool to compute m/z/RT deviation timelines and flag deviations
  > 20 ppm.
when_to_use_negative:
- You need drift correction or intensity-drift modeling rather than compound tracking
  — QC-MXP-style postanalysis corrections are a different module
- The input is a processed peak/feature table rather than vendor-raw or mzML files
  obtained directly from the instrument
- Your goal is contaminant screening across ~800 known ions — use module_Contaminants
  instead of a bespoke target list
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: Framework whose module_TrackCmp performs the peak detection and diagnostic
    extraction on the target m/z/RT list; results are stored and shown via the Shiny
    dashboard
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Implementation language; QC4Metabolomics leverages the rich ecosystem of R
    packages for the tracking modules
- name: Shiny
  role: Web dashboard that presents the per-injection tracked-compound results in
    (near) real time or retrospectively
- name: Docker
  role: Runs the ProteoWizard conversion that produces the mzML inputs consumed by
    module_TrackCmp, keeping the pipeline OS-agnostic
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:2c0c8fc15cc78478e1e2901e3172588c184836e682dfb18237ec8448c23b0925
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# qc-compound-peak-detection

## Summary

Detects and extracts chromatographic peaks for user-specified target compounds (m/z + RT pairs) from mzML injections, producing per-injection diagnostics (observed m/z, RT, intensity, peak shape) used to track instrument quality over time in QC4Metabolomics.

## When to use

Use when untargeted MS-based metabolomics mzML files (including pooled QC injections) must be monitored for a defined compound list — e.g. tracking the MetNexs long-term QC pool to compute m/z/RT deviation timelines and flag deviations > 20 ppm.

## When NOT to use

- You need drift correction or intensity-drift modeling rather than compound tracking — QC-MXP-style postanalysis corrections are a different module
- The input is a processed peak/feature table rather than vendor-raw or mzML files obtained directly from the instrument
- Your goal is contaminant screening across ~800 known ions — use module_Contaminants instead of a bespoke target list

## Inputs

- Vendor raw data converted to mzML (via ProteoWizard in Docker), also mzXML/mzData
- User-defined list of m/z and retention time (RT) pairs for target compounds (e.g. MetNexs long-term QC pool analytes)
- QC4Metabolomics database entries created by module_Files/module_File_schedule/module_File_info (instrument name, ionization mode, sample name)

## Outputs

- Per-injection tracked-compound table: observed m/z, retention time, intensity, peak shape diagnostics
- m/z deviation timeline in ppm vs. target values, with entries flagged where deviation exceeds 20 ppm
- Web dashboard views of tracked-compound time series for real-time and retrospective QC

## How to apply

Run module_TrackCmp on mzML/mzXML/mzData files that module_Files has registered in the QC4Metabolomics database, supplying a user-defined list of target m/z and retention time pairs (run per instrument, e.g. separately for the Waters Synapt). For each injection the module detects the peak at each target pair and extracts observed m/z, retention time, intensity and peak shape, which are stored in the database and rendered on the Shiny web dashboard. Compute m/z deviations in ppm against the target m/z across the timeline; deviations larger than 20 ppm indicate serious analytical problems that should be addressed promptly (the threshold used in module_Warner). RT deviations are expected to stay below 0.05 min in the short term, so excursions beyond that also signal drift. Repeat detection for every newly acquired file so that near-real-time and retrospective views share the same tracked-compound time series.

## Related tools

- **QC4Metabolomics** (Framework whose module_TrackCmp performs the peak detection and diagnostic extraction on the target m/z/RT list; results are stored and shown via the Shiny dashboard) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Implementation language; QC4Metabolomics leverages the rich ecosystem of R packages for the tracking modules)
- **Shiny** (Web dashboard that presents the per-injection tracked-compound results in (near) real time or retrospectively)
- **Docker** (Runs the ProteoWizard conversion that produces the mzML inputs consumed by module_TrackCmp, keeping the pipeline OS-agnostic)

## Examples

```
R> QC4Metabolomics::module_TrackCmp(list_of_mz_rt_pairs, mzml_files)  # per-injection observed m/z, RT, intensity, peak shape
```

## Evaluation signals

- Each target m/z/RT pair yields a detected peak entry for every QC injection, with no missing rows for expected positives
- Computed ppm deviations for a healthy instrument cluster around 0 and stay well below the 20 ppm warning threshold used by module_Warner
- Short-term RT deviations remain below 0.05 min; batch-to-batch shifts up to roughly twice that are attributable to batch effects, not detection failure
- Observed intensity and peak-shape columns are populated for all injections, enabling tryptophan-style three-week intensity-decrease events to be visible
- Dashboard time series update as new mzML files arrive via module_Files, matching real-time and retrospective views

## Limitations

- Detection depends on accurate target m/z/RT pairs; a small systematic calibration offset (e.g. observed May/June 2022) will bias observed m/z for all tracked compounds
- Instruments will typically not alert the user or stop acquisition when calibration is inaccurate — this tracking is the compensating control
- Intermittent or complete loss of calibrant signal (malfunctioning calibrant pump) compromises calibration and can render extracted m/z unreliable
- RT deviations between batches can be at least twice the short-term (<0.05 min) deviation, so cross-batch comparisons must account for batch effects

## Evidence

- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [abstract] diagnostic extraction scope: "extracting diagnostic information such as observed m/z , retention time, intensity and peak shape"
- [results] 20 ppm threshold: [evidence span withheld — anchor status: not_found]
- [results] short-term RT expectation: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [results] no instrument alert: "The instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate"
- [abstract] dashboard presentation: "and presents the results on a web dashboard"
