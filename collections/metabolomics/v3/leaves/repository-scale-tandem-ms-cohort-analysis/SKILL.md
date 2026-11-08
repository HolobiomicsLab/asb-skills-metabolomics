---
name: repository-scale-tandem-ms-cohort-analysis
description: Use when you need to reanalyze public tandem MS (MS/MS) files at repository
  scale as cohorts — e.g. compute PCA of per-file GNPS chemical annotations from the
  ReDU/MSV000084206 identification database — rather than analyzing one dataset in
  isolation.
when_to_use_negative:
- The goal is supervised group comparison or metadata association testing — use ReDU's
  Group Comparator or Sample Information Associations (View Associations button) instead
  of unsupervised PCA.
- You need to assemble cohorts of files by metadata filters — use the ReDU File Selector
  with the Attribute Filters Panel rather than recomputing the global PCA.
- Input is raw or converted mzXML/mzML data not yet annotated — GNPS spectral library
  search must produce chemical annotations before PCA can be run.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ReDU
  role: source of the repository-wide chemical annotation matrix and metadata; hosts
    the PCA results being reproduced
  repo: https://github.com/mwang87/ReDU-MS2-GNPS
- name: GNPS
  role: performs MS2 spectral-library (product ion spectra) search that produces the
    chemical annotations fed into the PCA
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
- name: Emperor
  role: visualization tool for the PCA results, supporting PC axis selection, coloring
    by sample information, and point filtering
  repo: https://github.com/biocore/emperor
- name: MassIVE
  role: public MS data repository whose files ReDU indexes; file identifiers anchor
    each PCA row
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-020-0916-7
    title: 'ReDU: a framework to find and reanalyze public mass spectrometry data'
  source_content_hash: sha256:f3e844fdf7e23b24a670de2056404cfd41538937b665ac7e8a957dd824e3af46
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# repository-scale-tandem-ms-cohort-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce ReDU's unsupervised PCA over the repository-wide chemical-annotation matrix (GNPS spectral-library annotations per MassIVE file) and export per-file principal-component coordinates for interactive 3D exploration in Emperor.

## When to use

Use when you need to reanalyze public tandem MS (MS/MS) files at repository scale as cohorts — e.g. compute PCA of per-file GNPS chemical annotations from the ReDU/MSV000084206 identification database — rather than analyzing one dataset in isolation.

## When NOT to use

- The goal is supervised group comparison or metadata association testing — use ReDU's Group Comparator or Sample Information Associations (View Associations button) instead of unsupervised PCA.
- You need to assemble cohorts of files by metadata filters — use the ReDU File Selector with the Attribute Filters Panel rather than recomputing the global PCA.
- Input is raw or converted mzXML/mzML data not yet annotated — GNPS spectral library search must produce chemical annotations before PCA can be run.

## Inputs

- ReDU repository-wide chemical-annotation matrix mapping each MassIVE file to GNPS-annotated chemicals (from MSV000084206)
- ReDU sample information / metadata for coloring and filtering PCA points in Emperor
- Per-file GNPS MS2 spectral-library-search annotation results (e.g. task TSVs as in database/global_tasks.tsv)

## Outputs

- Per-file PCA coordinate table (e.g. PCA_coordinates.tsv) with one row per analyzed file and principal-component coordinate columns
- PCA coordinates loadable into Emperor for interactive 3D visualization

## How to apply

Start from the ReDU database (MSV000084206, accessible via GNPS), in which chemical annotation of every MassIVE file comes from GNPS MS2 spectral-library search of product ion spectra. Load the repository-wide matrix mapping each MassIVE file to the chemicals annotated in that file; note that the same chemical can carry multiple GNPS annotations because slight m/z or abundance variation matches different reference MS2 spectra. Restrict the matrix to annotated chemical features present across the files, then run an unsupervised PCA (e.g. scikit-learn PCA) to derive principal-component scores per file — PCA is applied in an unsupervised manner exactly as ReDU does. Export the per-file coordinates as a tabular score file (e.g. PCA_coordinates.tsv) formatted like the ReDU global PCA output; the ReDU README notes temp files for 'the global pca' are regenerated on autopopulate. Validate that the coordinates reproduce the PCA ReDU displays in Emperor, where users can change PC axes, color points by sample information, and filter plotted data points.

## Related tools

- **ReDU** (source of the repository-wide chemical annotation matrix and metadata; hosts the PCA results being reproduced) — https://github.com/mwang87/ReDU-MS2-GNPS
- **GNPS** (performs MS2 spectral-library (product ion spectra) search that produces the chemical annotations fed into the PCA) — https://github.com/CCMS-UCSD/GNPSDocumentation
- **Emperor** (visualization tool for the PCA results, supporting PC axis selection, coloring by sample information, and point filtering) — https://github.com/biocore/emperor
- **MassIVE** (public MS data repository whose files ReDU indexes; file identifiers anchor each PCA row)

## Examples

```
from sklearn.decomposition import PCA; coords = PCA(n_components=3).fit_transform(chemical_annotation_matrix); pd.DataFrame(coords, index=file_ids).to_csv('PCA_coordinates.tsv', sep='\t')
```

## Evaluation signals

- Output TSV parses and contains exactly one row per analyzed MassIVE file with principal-component coordinate columns
- Coordinate table reproduces the PCA that ReDU displays in Emperor (same clustering of files on equivalent PC axes)
- Number of annotated chemical features matches the ReDU Chemical Explorer's count of unique annotated chemicals
- Emperor plot loads the coordinate table and supports changing PC axes, coloring by sample information, and filtering points
- Removing database content and restarting ReDU autopopulates the annotation matrix before global PCA views are computed (README update procedure invariant)

## Limitations

- GNPS annotations via spectral reference matching are only MSI level 2 or level 3 (putative annotations), so PCA operates on non-definitive identifications
- The same chemical can have multiple GNPS annotations because slight MS2 spectral variation matches different reference spectra, inflating feature-level redundancy
- GNPS library search has a recommended limit of 5000 files (3000 for molecular networking), constraining batch-scale annotation
- Recently contributed data are not displayed immediately in ReDU — data are updated periodically

## Evidence

- [abstract] Chemical annotation is performed in GNPS by comparing MS2 spectra, specifically product ion spectra, with reference MS2 fragmentation patterns: [evidence span withheld — anchor status: not_found]
- [readme] Remove all untracked files in temp, this will be for the global pca: [evidence span withheld — anchor status: external_doc]
