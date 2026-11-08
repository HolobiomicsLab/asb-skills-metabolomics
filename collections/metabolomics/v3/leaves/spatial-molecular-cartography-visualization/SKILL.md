---
name: spatial-molecular-cartography-visualization
description: Use when LC–MS runs were acquired across spatially arranged samples (e.g.
  swabs of a 96-well plate, plant surfaces, keypads) and the goal is a molecular intensity
  map localized onto an image or 3D model of the sampled surface.
when_to_use_negative:
- Input is already a feature quantification table from another metabolomics tool (Optimus
  expects raw mzML/mzXML runs and its own OpenMS-based detection)
- Data are direct-infusion or non-spatial batch LC–MS runs with no spatial mapping
  goal — Optimus supports such data but 'ili mapping and this skill are unnecessary
- Definitive (MSI level-1) molecular identification is required — the workflow only
  provides putative mz-RT level-2 annotation and MS/MS validation is not provided
  in Optimus
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: Optimus
  role: KNIME workflow that performs OpenMS-based LC–MS feature detection, alignment,
    quantification, optional filtering, mz-RT annotation, and export of 'ili_input.csv
    for spatial mapping
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: Provides the state-of-the-art LC–MS feature detection and quantification algorithms
    employed inside Optimus
- name: KNIME
  role: Workflow management platform in which the Optimus pipeline is executed
- name: '''ili'
  role: Web application for interactive 2D/3D visualization of feature intensities
    mapped on an image or 3D model of the sampled surface
  repo: https://github.com/ili-toolbox/ili
- name: OptimusViewer
  role: Application for visualizing XIC plots, MS peaks and MS2 spectra of features
    detected with Optimus from the produced *.db file
  repo: https://github.com/alexandrovteam/OptimusViewer
- name: pyopenms
  role: Python dependency installed alongside Optimus (pip install six pandas protobuf
    pymspec pyopenms)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:0efec859a5c63fe646c673ae047fb4c1055cfb9021a2ba3c3efd9eb3dbeb530f
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# spatial-molecular-cartography-visualization

## Summary

Reproduce spatial molecular maps from spatially sampled LC–MS untargeted metabolomics data by running the Optimus KNIME/OpenMS workflow for feature detection, alignment and mz-RT annotation, then visualizing feature intensities in 2D/3D with the 'ili app (and inspecting features with OptimusViewer).

## When to use

Use when LC–MS runs were acquired across spatially arranged samples (e.g. swabs of a 96-well plate, plant surfaces, keypads) and the goal is a molecular intensity map localized onto an image or 3D model of the sampled surface.

## When NOT to use

- Input is already a feature quantification table from another metabolomics tool (Optimus expects raw mzML/mzXML runs and its own OpenMS-based detection)
- Data are direct-infusion or non-spatial batch LC–MS runs with no spatial mapping goal — Optimus supports such data but 'ili mapping and this skill are unnecessary
- Definitive (MSI level-1) molecular identification is required — the workflow only provides putative mz-RT level-2 annotation and MS/MS validation is not provided in Optimus

## Inputs

- LC–MS run files in mzML or mzXML format (one per spatial spot/sample)
- Experimental design file mapping runs to sample positions (rows/wells of the plate or map)
- Molecules-of-interest list (CSV, or exported from GNPS spectral library matching) with m/z and RT values
- Background image (e.g. PNG of a 96-well plate) plus a coordinates CSV mapping each sample to pixel positions
- Optional pooled QC, blank/control and replicate runs for filtering and normalization

## Outputs

- features_quantification_matrix.csv (feature x run intensity matrix)
- ili_input.csv consumed by the 'ili app
- 2D/3D spatial molecular maps rendered in 'ili, shareable as cartographical snapshots
- Heat map of feature intensities across samples and 3D PCA plot of samples
- Optimus *.db file for inspection in OptimusViewer

## How to apply

Load the mzML/mzXML runs into the Optimus KNIME workflow, which uses OpenMS FeatureFinderMetabo for feature detection in each run and then aligns and quantifies consensus features across all runs. Apply the optional Optimus filter steps as needed (blank exclusion, rare-feature removal, QC-reproducibility filtering), and annotate features by m/z–RT matching against a molecules-of-interest list (e.g. minoxidil m/z 210.136, trehalose 6-tripalmitate m/z 603.335), which yields MSI level-2 'putatively annotated compounds'. Export the 'ili_input.csv together with a background PNG plate image plus a coordinates CSV mapping each well/spot to pixel positions, then load these into 'ili to render the spatial heat map; cartographical snapshots can be saved to reproduce the visualization. For per-feature QC of XICs, MS peaks and MS2 spectra, open the Optimus-produced *.db file in OptimusViewer.

## Related tools

- **Optimus** (KNIME workflow that performs OpenMS-based LC–MS feature detection, alignment, quantification, optional filtering, mz-RT annotation, and export of 'ili_input.csv for spatial mapping) — https://github.com/MolecularCartography/Optimus
- **OpenMS** (Provides the state-of-the-art LC–MS feature detection and quantification algorithms employed inside Optimus)
- **KNIME** (Workflow management platform in which the Optimus pipeline is executed)
- **'ili** (Web application for interactive 2D/3D visualization of feature intensities mapped on an image or 3D model of the sampled surface) — https://github.com/ili-toolbox/ili
- **OptimusViewer** (Application for visualizing XIC plots, MS peaks and MS2 spectra of features detected with Optimus from the produced *.db file) — https://github.com/alexandrovteam/OptimusViewer
- **pyopenms** (Python dependency installed alongside Optimus (pip install six pandas protobuf pymspec pyopenms))

## Examples

```
pip install six pandas protobuf pymspec pyopenms
```

## Evaluation signals

- Spatial map reproduces the expected localization: e.g. minoxidil intensity restricted to rows C and G and trehalose 6-tripalmitate to row H of the 96-well phone-swab plate, with blank row A free of these features
- Optimus output files (features_quantification_matrix.csv, ili_input.csv, *.db) exist and the quantification matrix has runs as columns and consensus features as rows with numeric ID, m/z, RT, charge and annotation columns first
- Annotated features match the molecules-of-interest list within the chosen tolerances (paper-recommended ~20 ppm m/z for QTOF data, ~30 s RT for a 10-min UHPLC gradient), at MSI level-2 annotation
- Blank/QC filter steps remove features from control runs and non-reproducible features, and internal-standard intensity timelines show stable acquisition across the run series
- A saved cartographical snapshot reloads the map with identical visualization settings

## Limitations

- The protocol takes ~78 h for a molecular map of 100 spots, excluding reagent setup
- Annotations from mz-RT matching are putative (MSI level 2); MS/MS validation is needed and is currently not provided in Optimus
- Optimus needs 64-bit OS and sufficient RAM — 2 GB is minimal but generally not enough for datasets of about a hundred or more LC–MS runs; temporary files can occupy a few times the dataset size and are not deleted automatically
- 'ili requires a background image or 3D model plus coordinates mapping samples to positions; only the pre-rendered 2D plate PNG path is demonstrated for 96-well maps

## Evidence

- [readme] OptimusViewer usage: [evidence span withheld — anchor status: external_doc]
