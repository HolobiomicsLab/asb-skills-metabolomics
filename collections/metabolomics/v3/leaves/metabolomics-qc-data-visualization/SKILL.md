---
name: metabolomics-qc-data-visualization
description: Use when monitoring untargeted MS-based metabolomics acquisitions — e.g.
  pooled QC injections every 5–10 injections — and you need to visualize RT/m/z deviation,
  intensity drift, or contaminant levels per batch to catch analytical problems.
when_to_use_negative:
- Input is already a processed feature table or post-corrected intensities (e.g. from
  QC-MXP) rather than raw vendor files needing real-time tracking.
- The goal is postanalysis intensity drift correction rather than monitoring/visualization
  — QC4Metabolomics tracks and visualizes QC parameters, it does not itself correct
  drifts.
- No pooled QC samples or user-defined m/z/RT pairs exist, so module_TrackCmp has
  no targets to monitor.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: open-source Shiny web application and R package that performs the file tracking,
    compound/contaminant extraction, and dashboard visualization for this skill
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: framework in which the web dashboard of QC results is rendered
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: container deployment (including docker-compose and ProteoWizard conversion)
    so the dashboard runs OS-agnostic on the instrument PC or a server
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: the underlying R package ecosystem that implements module_Files, module_TrackCmp,
    module_Contaminants, and module_Warner
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: ProteoWizard
  role: converts vendor-specific raw data to mzML before visualization in the dashboard
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:3db05aa87abcac1fed642e48b1beed32bcb7e88402bc56f2ca845583e493115f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - ProteoWizard
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabolomics-qc-data-visualization

## Summary

Presents QC4Metabolomics extraction and quality-control results (observed m/z, retention time, intensity, peak shape, and contaminant levels) on an interactive web dashboard so that drifts, calibration offsets, and instrument problems can be spotted in real time or retrospectively.

## When to use

Use when monitoring untargeted MS-based metabolomics acquisitions — e.g. pooled QC injections every 5–10 injections — and you need to visualize RT/m/z deviation, intensity drift, or contaminant levels per batch to catch analytical problems.

## When NOT to use

- Input is already a processed feature table or post-corrected intensities (e.g. from QC-MXP) rather than raw vendor files needing real-time tracking.
- The goal is postanalysis intensity drift correction rather than monitoring/visualization — QC4Metabolomics tracks and visualizes QC parameters, it does not itself correct drifts.
- No pooled QC samples or user-defined m/z/RT pairs exist, so module_TrackCmp has no targets to monitor.

## Inputs

- mzML, mzXML, or mzData files (converted from vendor raw data via ProteoWizard)
- user-defined list of m/z and retention time (RT) pairs for target compounds
- pooled QC sample injections placed at regular intervals among biological samples
- list of known contaminant ions (module_Contaminants uses nearly 800 entries)
- user-defined quality-parameter thresholds for module_Warner

## Outputs

- web-dashboard visualizations of QC parameters (observed m/z, RT, intensity, peak shape) per file/batch
- per-compound RT and m/z deviation tables with batch labels (e.g. MetNexs long-term QC pool)
- email warnings when a quality parameter exceeds a user-defined threshold
- contaminant level trend plots across the acquisition
- calendar/productivity overview and log of module activities

## How to apply

Run module_Files to pick up newly acquired mzML/mzXML/mzData files, let module_File_info and module_TrackCmp extract observed m/z, retention time, intensity, and peak shape for a user-defined list of m/z and RT pairs (e.g. the MetNexs long-term QC pool), then load the resulting table in the Shiny dashboard and group by batch identifier to compare within-batch versus between-batch deviations. For contaminants, inspect module_Contaminants results built from a list of nearly 800 known contaminant ions. Configure module_Warner thresholds so that quality-parameter excursions (e.g. RT deviations normally less than 0.05 min within a batch, or interbatch differences at least twice as large) trigger email alerts. Validate the visualization by confirming that the dashboard reproduces known artifacts such as the May/June 2022 calibration offset and the June sensitivity drop of an order of magnitude.

## Related tools

- **QC4Metabolomics** (open-source Shiny web application and R package that performs the file tracking, compound/contaminant extraction, and dashboard visualization for this skill) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (framework in which the web dashboard of QC results is rendered) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (container deployment (including docker-compose and ProteoWizard conversion) so the dashboard runs OS-agnostic on the instrument PC or a server) — https://github.com/stanstrup/QC4Metabolomics
- **R** (the underlying R package ecosystem that implements module_Files, module_TrackCmp, module_Contaminants, and module_Warner) — https://github.com/stanstrup/QC4Metabolomics
- **ProteoWizard** (converts vendor-specific raw data to mzML before visualization in the dashboard)

## Examples

```
Rscript -e "QC4Metabolomics::module_TrackCmp(mz_rt_pairs = my_qc_pool, mzml_dir = 'raw_data/')"
```

## Evaluation signals

- Dashboard reproduces the article's reported within-batch RT deviations of less than 0.05 min for the MetNexs QC pool.
- Between-batch RT differences are at least twice as large as within-batch deviations, matching the reported two-regime result.
- Email alerts fire (module_Warner) when a quality parameter exceeds its user-defined threshold, e.g. during the June 2022 order-of-magnitude sensitivity drop.
- Contaminant trend panel reflects module_Contaminants output built from the ~800 known contaminant ions.
- module_Log and module_Productivity show the expected sequence of file additions and module activities for the batch.

## Limitations

- Instruments typically do not alert the user or stop the acquisition when calibration is inaccurate, so problems must be caught by monitoring this dashboard rather than the instrument itself.
- Some quality problems — e.g. loss of calibration in a limited mass region, nonobvious contaminants, or ion suppression/enhancement in specific chromatographic regions — are not obvious and require retrospective inspection of the dashboard.
- Calibration quality depends on a reliable calibrant signal; a malfunctioning calibrant pump (as observed at the end of 2022) can compromise calibration and thus the m/z deviation readout.

## Evidence

- [abstract] Monitoring QC parameters on a web dashboard: [evidence span withheld — anchor status: not_found]
- [intro] Real-time tracking of compounds: "monitors QC parameters in metabolomics experiments in (near) real-time"
- [methods] TrackCmp module: "module_TrackCmp detects peaks based
on a user-defined list of m/z and
retention time (RT) pairs"
- [results] RT deviation two-regime result: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [results] Real-world problem detection: [evidence span withheld — anchor status: not_found]
- [readme] README pointer: "Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/"
