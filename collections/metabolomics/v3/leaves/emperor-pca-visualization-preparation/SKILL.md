---
name: emperor-pca-visualization-preparation
description: Use when you have a per-file chemical annotation matrix (e.g. from GNPS
  library search over MassIVE/MSV000084206 files) and need unsupervised PCA coordinates
  formatted for Emperor visualization.
when_to_use_negative:
- Input is already a feature table of per-file annotation counts and you only need
  to run PCA — this skill covers matrix construction plus PCA plus Emperor-ready export,
  not just the PCA step
- The analysis is supervised group comparison (e.g. ReDU Group Comparator cohort plots)
  rather than unsupervised dimensionality reduction of chemical annotations
- Data are raw tandem MS files (.mzXML/.mzML) rather than a chemical annotation matrix
  — annotation must first be produced via GNPS spectral library search
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ReDU
  role: Source of the repository-wide chemical annotation matrix (per-file GNPS annotations)
    that serves as the PCA input, plus the upstream metadata used for Emperor coloring
  repo: https://github.com/mwang87/ReDU-MS2-GNPS
- name: GNPS
  role: Produces the MS2 spectral library annotations (chemical identifications) that
    populate the feature table before PCA
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
- name: Emperor
  role: Interactive 3D visualization of the PCA results; supports changing PC axes,
    coloring by sample information, and filtering plotted points
  repo: https://github.com/biocore/emperor
- name: MassIVE
  role: Repository whose public tandem MS data files constitute the rows of the annotation
    matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-020-0916-7
    title: 'ReDU: a framework to find and reanalyze public mass spectrometry data'
  source_content_hash: sha256:e1997bccaa4c76b3f0cf207d5d1c45168d90b9128e2bd247e01f2e6643b9d7c1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mwang87/ReDU-MS2-Documentation
  repo_url: https://github.com/mwang87/ReDU-MS2-GNPS
  tools:
  - Emperor
  - GNPS
  - MassIVE
  - ReDU
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# emperor-pca-visualization-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares a repository-scale PCA coordinate table from the chemical annotations ReDU derives via GNPS spectral library search, so that the principal-component results over public tandem MS data can be explored interactively in Emperor.

## When to use

Use when you have a per-file chemical annotation matrix (e.g. from GNPS library search over MassIVE/MSV000084206 files) and need unsupervised PCA coordinates formatted for Emperor visualization.

## When NOT to use

- Input is already a feature table of per-file annotation counts and you only need to run PCA — this skill covers matrix construction plus PCA plus Emperor-ready export, not just the PCA step
- The analysis is supervised group comparison (e.g. ReDU Group Comparator cohort plots) rather than unsupervised dimensionality reduction of chemical annotations
- Data are raw tandem MS files (.mzXML/.mzML) rather than a chemical annotation matrix — annotation must first be produced via GNPS spectral library search

## Inputs

- Repository-wide chemical annotation matrix mapping each MassIVE file to the GNPS-annotated chemicals found in that file (MS2 product ion spectral library matches)

## Outputs

- Per-file PCA coordinate table (e.g. PCA_coordinates.tsv) with principal-component score columns for each analyzed file, formatted for Emperor upload
- Interactive 3D principal-component visualization in Emperor with metadata coloring and point filtering

## How to apply

Build a feature table whose rows are MassIVE files and whose columns are the chemical annotations from GNPS spectral library search, counting occurrences of each annotated chemical per file. Run PCA on this annotation matrix in an unsupervised manner (e.g. sklearn.decomposition.PCA.fit) to obtain principal component scores for every file, mirroring how ReDU analyzes the chemicals annotated per file repository-wide. Export the per-file PC coordinates as a tabular score-plot file (e.g. PCA_coordinates.tsv) with one row per analyzed file and PC coordinate columns, so it can be uploaded into Emperor. In Emperor, users can then change PC axes, color points by sample information (metadata), and filter plotted data points. Remember that the same chemical can have multiple GNPS annotations due to slight variation in MS2 spectra (m/z or abundance), so the feature table will contain multiple related annotation columns rather than a single canonical chemical per feature.

## Related tools

- **ReDU** (Source of the repository-wide chemical annotation matrix (per-file GNPS annotations) that serves as the PCA input, plus the upstream metadata used for Emperor coloring) — https://github.com/mwang87/ReDU-MS2-GNPS
- **GNPS** (Produces the MS2 spectral library annotations (chemical identifications) that populate the feature table before PCA) — https://github.com/CCMS-UCSD/GNPSDocumentation
- **Emperor** (Interactive 3D visualization of the PCA results; supports changing PC axes, coloring by sample information, and filtering plotted points) — https://github.com/biocore/emperor
- **MassIVE** (Repository whose public tandem MS data files constitute the rows of the annotation matrix)

## Examples

```
from sklearn.decomposition import PCA; coords = PCA(n_components=3).fit_transform(chem_annotation_matrix); coords.to_csv('PCA_coordinates.tsv', sep='\t')
```

## Evaluation signals

- Output coordinate file is a parseable TSV with exactly one row per analyzed file
- Each row contains principal-component coordinate columns corresponding to the unsupervised PCA over the chemical-annotation feature table
- The number of files represented matches the file count expected from the ReDU File Selector / MassIVE corpus used
- Axis labels / columns correspond to PC1..PCn and can be swapped in Emperor as users change PC axes
- Coloring dimensions in Emperor map correctly onto ReDU sample information (metadata) fields

## Limitations

- Recommended limits: 3000 files for GNPS molecular networking and 5000 for GNPS library search, so very large repository-scale matrices may be constrained by GNPS workflow limits
- Data updates are periodic; recently contributed data will not display immediately in ReDU
- Peer-reviewed publication status was pending at the time the documentation was written
- The same chemical can have multiple GNPS annotations (spectral variants), inflating feature columns relative to distinct chemicals

## Evidence
