---
name: contaminant-peak-detection
description: Use when monitoring background contaminants (e.g. from solvents or sample
  carryover) across pooled-QC injections in an mzML-based untargeted metabolomics
  run, where you want time-series, per-sample composition, or heatmap views of contaminant
  levels.
when_to_use_negative:
- If your goal is general feature detection and peak picking on individual biological
  samples rather than tracking known background contaminants across a run — module_TrackCmp
  with a user-defined compound list is the appropriate module instead.
- If the input data are not untargeted MS-based metabolomics files (e.g. targeted
  MRM, imaging MS, or NMR), the contaminant list matching against RT/m/z pairs is
  not applicable.
- If you need to correct intensity drifts rather than monitor their presence, QC-MXP
  provides tools for correcting intensity drifts and should be used instead.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: QC4Metabolomics
  role: Real-time and retrospective QC framework in which module_Contaminants runs
    the contaminant peak detection and stores results in mariaDB for dashboard rendering
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Web dashboard that presents the three contaminant-monitoring views (time trend,
    sample composition barplot, heatmap)
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Containerized deployment via docker-compose; environment variables pass the
    contaminant list to the module_Contaminants container
- name: R
  role: Implementation language of the QC4Metabolomics R package and Shiny application
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:8f37fdad7cc138331c64946f0565d38af2f030db6891b5281a7f48375b4e59ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# contaminant-peak-detection

## Summary

Detects contaminant peaks in untargeted LC-MS metabolomics files by matching observed peaks against a curated list of nearly 800 known contaminant m/z and retention-time pairs, storing diagnostics (observed m/z, retention time, intensity, peak shape) in a mariaDB database and surfacing them on a Shiny web dashboard.

## When to use

Use when monitoring background contaminants (e.g. from solvents or sample carryover) across pooled-QC injections in an mzML-based untargeted metabolomics run, where you want time-series, per-sample composition, or heatmap views of contaminant levels.

## When NOT to use

- If your goal is general feature detection and peak picking on individual biological samples rather than tracking known background contaminants across a run — module_TrackCmp with a user-defined compound list is the appropriate module instead.
- If the input data are not untargeted MS-based metabolomics files (e.g. targeted MRM, imaging MS, or NMR), the contaminant list matching against RT/m/z pairs is not applicable.
- If you need to correct intensity drifts rather than monitor their presence, QC-MXP provides tools for correcting intensity drifts and should be used instead.

## Inputs

- mzML / mzXML / mzData files acquired during an untargeted LC-MS metabolomics run (e.g. pooled-QC injections)
- A list of known contaminant ions as m/z and retention-time pairs (the bundled ~800-ion list or a user-supplied list)

## Outputs

- Per-contaminant diagnostic records in mariaDB: observed m/z, retention time, intensity, and peak shape for each contaminant detection
- Three contaminant-monitoring views on the Shiny dashboard: contaminant level over time, per-sample contaminant composition barplot, and a heatmap clustering contaminants over analysis time

## How to apply

Deploy QC4Metabolomics via the provided docker-compose setup (optionally with demo data). Enable module_Contaminants and supply either the bundled list of ~800 known contaminant ions or a user-supplied list of m/z and retention-time pairs via environment variables passed to the container. Let module_Files detect newly acquired files (mzML, mzXML, mzData) by traversing a directory or reading from a text file, and module_File_schedule queue them for processing. Run module_Contaminants, which detects contaminant peaks from the m/z and RT pairs in the contaminant list and stores diagnostic information (observed m/z, retention time, intensity, peak shape) in the mariaDB database. Finally, query the database and build the three contaminant-monitoring views for the Shiny dashboard: a time view plotting contaminant changes over time, a per-sample composition barplot sorting contaminants by intensity, and a heatmap clustering contaminants over analysis time.

## Related tools

- **QC4Metabolomics** (Real-time and retrospective QC framework in which module_Contaminants runs the contaminant peak detection and stores results in mariaDB for dashboard rendering) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Web dashboard that presents the three contaminant-monitoring views (time trend, sample composition barplot, heatmap)) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Containerized deployment via docker-compose; environment variables pass the contaminant list to the module_Contaminants container)
- **R** (Implementation language of the QC4Metabolomics R package and Shiny application) — https://github.com/stanstrup/QC4Metabolomics

## Examples

```
docker-compose up -d  # starts the QC4Metabolomics stack; module_Contaminants runs against the bundled ~800 contaminant ions from env vars
```

## Evaluation signals

- All three contaminant views (time trend, sample composition, heatmap) render on the Shiny dashboard for the demo data.
- Every detected contaminant can be traced to an entry in the ~800-ion contaminant list (observed m/z and RT match the supplied pairs).
- Diagnostic fields (observed m/z, retention time, intensity, peak shape) are populated and non-null in the mariaDB table for each detection.
- module_Files has registered the newly acquired mzML/mzXML/mzData files and module_File_schedule has queued them, so no file is silently skipped.
- Contaminant intensities are consistent in magnitude and rank ordering with the per-sample composition barplot (contaminants sorted by intensity).

## Limitations

- The bundled contaminant list covers nearly 800 known ions; contaminants outside this list must be supplied by the user, so novel or nonobvious contaminants are missed.
- Detection relies on matching a user-defined m/z and retention-time pair; loss of calibration in a limited mass region or RT drifts (which can be at least twice as large between batches) can cause mis-mapping.
- The instruments typically do not alert the user or stop acquisition if the calibration is inaccurate, so contaminant detection quality depends on upstream instrument health that QC4Metabolomics monitors retrospectively.

## Evidence

- [methods] module_Contaminants uses a list of nearly 800 known contaminant ions: "module_Contaminants uses a list of nearly 800 known contaminant ions"
- [abstract] extracting diagnostic information such as observed m/z , retention time, intensity and peak shape: "extracting diagnostic information such as observed m/z , retention time, intensity and peak shape"
- [abstract] Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data.: "Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data."
- [methods] module_Files looks for newly acquired files (mzML, mzXML, mzData etc.): "module_Files looks for newly acquired files (mzML, mzXML, mzData etc.)"
