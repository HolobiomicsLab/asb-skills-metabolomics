---
name: lc-ms-feature-detection
description: Use when centroided mzML LC-MS/MS runs (e.g. QTOF or Orbitrap spatial-mapping
  datasets) must be converted into a feature quantification matrix as the entry point
  of an Optimus/'ili cartography analysis.
when_to_use_negative:
- Input is already an aligned feature quantification table (proceed directly to annotation,
  filtering, or 'ili input generation instead).
- The analysis goal is targeted validation of a known compound list rather than untargeted
  feature detection across runs.
- The data are not LC-MS runs in centroided mzML form (e.g. profile-mode vendor files
  that have not been converted).
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Optimus
  role: KNIME workflow hosting the 'Read LC-MS runs', 'Detect LC-MS features', and
    'Align and quantify features' nodes
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: provides the state-of-the-art LC-MS feature detection and quantification algorithms
    (FeatureFinderMetabo) used by Optimus
  repo: http://www.openms.de
- name: KNIME
  role: workflow management system that joins the OpenMS detection and alignment steps
    into the Optimus pipeline
  repo: https://www.knime.org
- name: OptimusViewer
  role: application to inspect detected features as XIC plots, MS peaks, and MS2 spectra
    from the Optimus .db output
  repo: https://github.com/alexandrovteam/OptimusViewer
- name: pyopenms
  role: Python module installed as an Optimus dependency for KNIME-Python integration
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:70b6434724c777a09bb1c91fc9a586c82abbab5a0969172c676c8d47e4d93b2d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MolecularCartography/Optimus
  tools:
  - '''ili'
  - KNIME
  - OpenMS
  - Optimus
  - OptimusViewer
  - Python
  - pyopenms
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# lc-ms-feature-detection

## Summary

Detect LC-MS features in centroided mzML runs using the OpenMS-based feature-detection nodes of the Optimus KNIME workflow, the first step of the 3D molecular cartography pipeline. It converts raw instrument runs into a quantified, aligned feature table that downstream filtering, annotation, and 'ili spatial mapping build upon.

## When to use

Use when centroided mzML LC-MS/MS runs (e.g. QTOF or Orbitrap spatial-mapping datasets) must be converted into a feature quantification matrix as the entry point of an Optimus/'ili cartography analysis.

## When NOT to use

- Input is already an aligned feature quantification table (proceed directly to annotation, filtering, or 'ili input generation instead).
- The analysis goal is targeted validation of a known compound list rather than untargeted feature detection across runs.
- The data are not LC-MS runs in centroided mzML form (e.g. profile-mode vendor files that have not been converted).

## Inputs

- centroided mzML LC-MS/MS run files retrieved from MassIVE (e.g. MSV000080553, MSV000080560, MSV000078556)
- optimus_experimental_design.csv marking BLANK and pooled QC runs
- Optimus stub input file configuration

## Outputs

- features_quantification_matrix.csv (feature ID, m/z, retention time, charge, per-run intensities)
- OptimusViewer_input.db (database of detected features loadable in OptimusViewer)
- Optimus_settings.ini (recorded workflow parameters)
- spectra_ms2.mgf (MS2 spectra of detected features for downstream tools)

## How to apply

Load the centroided mzML files into Optimus's 'Read LC-MS runs' node in KNIME and generate the experimental design template, marking blank and QC runs in optimus_experimental_design.csv. Configure the 'Detect LC-MS features' node (OpenMS FeatureFinderMetabo) with an m/z tolerance of 20 p.p.m. for QTOF data (5–10 p.p.m. for Orbitrap) and a QTOF noise threshold of 5,000 counts (100,000–1,000,000 for Orbitrap). Downstream in 'Align and quantify features', set the RT tolerance to 30 s, the default for a 10-min UHPLC gradient. Execute the workflow to align and quantify features across all runs; the parameter choices are recorded in Optimus_settings.ini for reproducibility. The detection step feeds all subsequent optional steps (blank filtering, annotation, normalization, spatial map export), so detection should be run and iterated before enabling filters.

## Related tools

- **Optimus** (KNIME workflow hosting the 'Read LC-MS runs', 'Detect LC-MS features', and 'Align and quantify features' nodes) — https://github.com/MolecularCartography/Optimus
- **OpenMS** (provides the state-of-the-art LC-MS feature detection and quantification algorithms (FeatureFinderMetabo) used by Optimus) — http://www.openms.de
- **KNIME** (workflow management system that joins the OpenMS detection and alignment steps into the Optimus pipeline) — https://www.knime.org
- **OptimusViewer** (application to inspect detected features as XIC plots, MS peaks, and MS2 spectra from the Optimus .db output) — https://github.com/alexandrovteam/OptimusViewer
- **pyopenms** (Python module installed as an Optimus dependency for KNIME-Python integration)

## Evaluation signals

- The workflow completes and produces features_quantification_matrix.csv together with Optimus_settings.ini, OptimusViewer_input.db, and spectra_ms2.mgf.
- The detection settings recorded in Optimus_settings.ini show m/z tolerance 20 p.p.m. and noise threshold 5,000 for QTOF data (or 5–10 p.p.m. and 100,000–1,000,000 for Orbitrap), and RT tolerance 30 s in the alignment node.
- OptimusViewer loads the resulting .db file and shows the feature quantification matrix with numeric feature ID, m/z, retention time, charge, and annotation columns.
- Features detected in individual runs align across all runs within the configured 30 s RT tolerance in 'Align and quantify features'.
- Features with acquired MS/MS spectra are exported to spectra_ms2.mgf, enabling downstream GNPS or in-silico (Sirius/MS-FINDER) analysis.

## Limitations

- Only 64-bit systems are supported; ~2 GB RAM is minimal but insufficient for datasets of about a hundred or more LC-MS runs.
- Temporary files can occupy up to a few times the initial dataset size and are not deleted automatically (they are kept to enable iterative re-execution).
- The detection step is untargeted; MS/MS validation of putative mz-RT annotations is not provided within Optimus.
- Optimal m/z tolerance and noise threshold depend on the instrument: 20 p.p.m./5,000 counts are QTOF recommendations, not universal defaults.

## Evidence
