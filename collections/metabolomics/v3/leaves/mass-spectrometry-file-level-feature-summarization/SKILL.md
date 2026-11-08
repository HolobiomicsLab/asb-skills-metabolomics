---
name: mass-spectrometry-file-level-feature-summarization
description: Use when a collection of public LC-MS/MS files (e.g. those indexed in
  ReDU / MassIVE MSV000084206) needs an unsupervised, file-level overview of chemical
  content, or when PCA score plots colored by sample information are needed to explore
  repository-scale chemical annotations.
when_to_use_negative:
- Input is already per-feature intensities within a single sample rather than file-level
  chemical annotations across many files — PCA here summarizes files, not features
  within one dataset
- A supervised or metadata-driven grouping is desired; ReDU's PCA is explicitly unsupervised
  and does not use sample information to derive the components (metadata is only applied
  afterward for coloring/filtering in Emperor)
- The goal is de novo annotation of spectra rather than summarizing existing GNPS
  library-search annotations — chemical annotation itself is a separate GNPS spectral
  library search step
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ReDU
  role: Source of the file-to-chemical annotation matrix and sample information; hosts
    the global PCA that this skill reproduces
  repo: https://github.com/mwang87/ReDU-MS2-GNPS
- name: GNPS
  role: Performs the MS2 product ion spectral library search that produces the per-file
    chemical annotations used as PCA features
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
- name: Emperor
  role: Interactive visualization of the per-file PCA coordinates; supports changing
    PC axes, coloring by sample information, and filtering plotted points
  repo: https://github.com/biocore/emperor
- name: MassIVE
  role: Public repository supplying the raw MS/MS files indexed by ReDU; each file
    is keyed by its MassIVE ID in the annotation matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-020-0916-7
    title: 'ReDU: a framework to find and reanalyze public mass spectrometry data'
  source_content_hash: sha256:d3d1673c69fa383f657543d0de28ac4cb40bd0898f21bc6f1d6782eb3c8a7c89
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
yes_rate: 1.0
n_claims: 10
---

# mass-spectrometry-file-level-feature-summarization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Summarizes repository-scale tandem MS data at the file level by building a matrix mapping each MassIVE file to the chemicals annotated in it via GNPS spectral library search, then applying unsupervised PCA over that chemical-annotation feature table and visualizing the principal component scores in Emperor.

## When to use

Use when a collection of public LC-MS/MS files (e.g. those indexed in ReDU / MassIVE MSV000084206) needs an unsupervised, file-level overview of chemical content, or when PCA score plots colored by sample information are needed to explore repository-scale chemical annotations.

## When NOT to use

- Input is already per-feature intensities within a single sample rather than file-level chemical annotations across many files — PCA here summarizes files, not features within one dataset
- A supervised or metadata-driven grouping is desired; ReDU's PCA is explicitly unsupervised and does not use sample information to derive the components (metadata is only applied afterward for coloring/filtering in Emperor)
- The goal is de novo annotation of spectra rather than summarizing existing GNPS library-search annotations — chemical annotation itself is a separate GNPS spectral library search step

## Inputs

- Repository-wide chemical annotation matrix mapping each MassIVE file (from MSV000084206, the ReDU identification database) to chemicals annotated by GNPS MS2 spectral library search
- GNPS spectral library search results (batch spectral matching task outputs, e.g. global_tasks.tsv) for all files in ReDU
- ReDU sample information / metadata for coloring and filtering points in Emperor

## Outputs

- Per-file principal component coordinate table (e.g. PCA_coordinates.tsv), one row per analyzed file with PC score columns
- Interactive Emperor visualization of the PCA score plot with PC axis selection, sample-information coloring, and data point filtering

## How to apply

Load the repository-wide chemical annotation matrix that maps each MassIVE file to the chemicals annotated in that file by GNPS MS2 spectral library search (product ion spectra compared against reference MS2 fragmentation patterns), restricted to the annotated chemical features across all files in the ReDU identification database (MSV000084206). Note that the same chemical can have multiple GNPS annotations due to slight variation in MS2 spectra, so the matrix rows are per-file spectral matches rather than unique compounds. Run unsupervised PCA (e.g. scikit-learn PCA) on this binary/feature table to derive principal component scores for each file, mirroring ReDU's own global PCA computation which is regenerated after database updates. Export the per-file PCA coordinates as a tabular file (e.g. PCA_coordinates.tsv) suitable for upload into Emperor, where users can change PC axes, color points by sample information, and filter plotted data points. Correctness is judged by the score table containing one row per analyzed file with PC coordinate columns and by the PCA reproducing the structure ReDU displays in Emperor.

## Related tools

- **ReDU** (Source of the file-to-chemical annotation matrix and sample information; hosts the global PCA that this skill reproduces) — https://github.com/mwang87/ReDU-MS2-GNPS
- **GNPS** (Performs the MS2 product ion spectral library search that produces the per-file chemical annotations used as PCA features) — https://github.com/CCMS-UCSD/GNPSDocumentation
- **Emperor** (Interactive visualization of the per-file PCA coordinates; supports changing PC axes, coloring by sample information, and filtering plotted points) — https://github.com/biocore/emperor
- **MassIVE** (Public repository supplying the raw MS/MS files indexed by ReDU; each file is keyed by its MassIVE ID in the annotation matrix)

## Examples

```
from sklearn.decomposition import PCA; scores = PCA(n_components=3).fit_transform(file_by_chemical_matrix); scores.to_csv('PCA_coordinates.tsv', sep='\t')
```

## Evaluation signals

- Output table exists and is a parseable TSV containing one row per analyzed MassIVE file with principal-component coordinate columns
- PCA is unsupervised: no sample-information columns are used as PCA inputs, only the file-by-chemical annotation matrix
- Number of unique chemicals matches what ReDU's Chemical Explorer reports for the database (per ReDU's manual validation checklist)
- Emperor accepts the exported coordinate table and renders an interactive 3D ordination with working axis switching, sample-information coloring, and point filtering
- Result reproduces the unsupervised PCA that ReDU itself displays in Emperor after its data autopopulation completes

## Limitations

- GNPS spectral library annotations are only level 2/3 (putative annotations) under the 2007 metabolomics standards initiative, so PCA structure reflects putative chemical content
- The same chemical can carry multiple GNPS annotations due to slight m/z or abundance variation in MS2 spectra, inflating feature columns relative to unique compounds
- Recommended limits are 5000 files for GNPS library search (3000 for molecular networking), bounding how many files can be annotated per batch
- ReDU data are updated periodically and will not display recently contributed data immediately; the global PCA should only be generated after the database fully autopopulates

## Evidence

- [abstract] Chemical annotation is performed in GNPS by comparing MS2 spectra, specifically product ion spectra, with reference MS2 fragmentation patterns: [evidence span withheld — anchor status: not_found]
- [readme] Remove all untracked files in temp, this will be for the global pca ... Start ReDU back up and it will autopopulate ... Wait Until full autopopulate before going to global PCA views: [evidence span withheld — anchor status: external_doc]
- [readme] Check that graphs in Chemical Explorer are functional, available after clicking an example chemical's 'view assocations' button.: [evidence span withheld — anchor status: external_doc]
