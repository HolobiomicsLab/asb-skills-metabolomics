---
name: metabolomics-quality-parameter-extraction
description: Use when acquired or archived untargeted MS metabolomics files (mzML,
  mzXML, mzData) must be summarized into per-file quality metrics — RT and m/z deviations,
  intensity drift, peak shape — for real-time or retrospective QC.
when_to_use_negative:
- Input is already an annotated feature table or processed peak matrix — this skill
  operates on raw mzML/mzXML/mzData acquisition files.
- The goal is postanalysis intensity-drift correction rather than per-file QC parameter
  extraction — use tools like QC-MXP instead.
- The analysis is a targeted assay with no pooled QC samples injected at regular intervals
  (every 5–10 injections) — the tracked-compound approach presumes such QC samples
  exist.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: Provides the Shiny app and R package whose module_File_info and module_TrackCmp
    modules perform metadata extraction and compound tracking on mzML files
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Web-application framework that hosts the dashboard where extracted quality
    parameters are presented in real time or retrospectively
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Packaging and deployment mechanism; also runs ProteoWizard for vendor raw-data
    to mzML conversion
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Language of the underlying package; QC4Metabolomics leverages the rich ecosystem
    of R packages for peak detection and feature extraction
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:4546be57d76638bb0897fbd6256137c90fcb0e48c22ae756e2ab6a8ba632c5f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolomics-quality-parameter-extraction

## Summary

Extract per-file quality parameters from untargeted metabolomics mzML data by tracking user-defined compounds (observed m/z, retention time, intensity, peak shape) and metadata, as implemented in QC4Metabolomics's module_File_info and module_TrackCmp.

## When to use

Use when acquired or archived untargeted MS metabolomics files (mzML, mzXML, mzData) must be summarized into per-file quality metrics — RT and m/z deviations, intensity drift, peak shape — for real-time or retrospective QC.

## When NOT to use

- Input is already an annotated feature table or processed peak matrix — this skill operates on raw mzML/mzXML/mzData acquisition files.
- The goal is postanalysis intensity-drift correction rather than per-file QC parameter extraction — use tools like QC-MXP instead.
- The analysis is a targeted assay with no pooled QC samples injected at regular intervals (every 5–10 injections) — the tracked-compound approach presumes such QC samples exist.

## Inputs

- Vendor raw data converted to mzML (also accepts mzXML, mzData) copied to a network drive accessible to QC4Metabolomics
- User-defined list of m/z and retention time (RT) pairs for compounds to track
- Instrument PC batch script output: raw files copied to the watched directory or a text file listing file paths

## Outputs

- Database table of per-file metadata (instrument name, project name, ionization mode, sample name)
- Per-file tracked-compound records: observed m/z, retention time, intensity, peak shape diagnostics
- Per-file processing timing (about 14 s per file on the bundled demo data)

## How to apply

Deploy QC4Metabolomics (Docker/docker-compose) and run the continuously processing modules on newly acquired files. First, module_Files traverses the acquisition directory (or a text file of paths) and adds mzML/mzXML/mzData files to the database; module_File_schedule then schedules new filenames for processing. module_File_info extracts metadata (instrument name, project name, ionization mode, sample name), while module_TrackCmp detects peaks against a user-defined list of m/z and retention time pairs, recording observed m/z, RT, intensity and peak-shape diagnostics. Validation: on the bundled demo data, extraction takes about 14 s per file, so compare measured timings and confirm tracked compounds yield non-empty peak detections with plausible RT (< 0.05 min deviation in the short term) and m/z values consistent with calibration.

## Related tools

- **QC4Metabolomics** (Provides the Shiny app and R package whose module_File_info and module_TrackCmp modules perform metadata extraction and compound tracking on mzML files) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Web-application framework that hosts the dashboard where extracted quality parameters are presented in real time or retrospectively) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Packaging and deployment mechanism; also runs ProteoWizard for vendor raw-data to mzML conversion) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Language of the underlying package; QC4Metabolomics leverages the rich ecosystem of R packages for peak detection and feature extraction) — https://github.com/stanstrup/QC4Metabolomics

## Examples

```
docker-compose up -d   # QC4Metabolomics with demo data; module_Files → module_File_schedule → module_File_info → module_TrackCmp extract m/z, RT, intensity, peak shape per mzML file
```

## Evaluation signals

- Each mzML file appears in the database and is scheduled by module_File_schedule, and metadata (instrument name, project name, ionization mode, sample name) is populated by module_File_info.
- For each user-defined m/z/RT pair, module_TrackCmp returns a detected peak with observed m/z, RT, intensity, and peak-shape metrics — no tracked compound should be silently missing.
- On the bundled demo data, quality-parameter extraction should take about 14 s per file (machine-dependent absolute times).
- Short-term RT deviations for tracked compounds should normally be less than 0.05 min; batch-to-batch differences may be at least twice as large, so deviations above this flag calibration issues.

## Limitations

- Absolute extraction times depend on the host machine; the ~14 s/file figure is specific to the demo data and hardware.
- Extraction only covers compounds present in the user-defined m/z/RT tracking list — analytes outside this list yield no quality parameters.
- Loss of calibration in a limited mass region, nonobvious contaminants, or ion suppression affecting only certain peaks will not be detected by the tracked-compound extraction alone.

## Evidence

- [results] extraction of quality parameters takes about 14 s per file: [evidence span withheld — anchor status: not_found]
- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [methods] module_File_info extracts information (e.g., instrument name, project name, ionization mode and sample name): "module_File_info extracts information (e.g., instrument name, project name, ionization mode and sample name)"
- [results] in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [methods] module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths: "module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths"
