---
name: chemical-annotation-matrix-pca-analysis
description: Use when you have a file-by-chemical annotation table (GNPS spectral
  library matches per MassIVE file) and want to reproduce ReDU's repository-scale,
  unsupervised PCA ordination of public tandem MS files.
when_to_use_negative:
- Input is already a PC-coordinate table or an existing Emperor mapping file — go
  straight to visualization instead of recomputing PCA
- The goal is supervised or hypothesis-driven comparison of cohorts — use ReDU's Group
  Comparator or metadata-filtered molecular networking rather than unsupervised PCA
- Data have not been chemically annotated (no GNPS library-search results per file)
  — there is no chemical-annotation feature table to ordinate
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: ReDU
  role: Source of the per-file chemical annotation matrix and sample information;
    provides the identification database (MSV000084206) whose files define the analysis
    universe
  repo: https://github.com/mwang87/ReDU-MS2-GNPS
- name: GNPS
  role: Performs the MS2 spectral library search that produces the chemical annotations
    (feature table) ordinated by PCA
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
- name: Emperor
  role: 'Interactive visualization of the PCA results: change PC axes, color points
    by sample information, and filter plotted data points'
  repo: https://github.com/biocore/emperor
- name: MassIVE
  role: Public repository whose file identifiers (MassIVE IDs) index rows of the annotation
    matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-020-0916-7
    title: 'ReDU: a framework to find and reanalyze public mass spectrometry data'
  source_content_hash: sha256:6049f437ad0b43b8188b0596b712dc460f6d565b05ceef97435b4f8b4e3feea0
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# chemical-annotation-matrix-pca-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces ReDU's unsupervised principal components analysis of the per-file GNPS chemical annotation matrix (from MSV000084206) and exports per-file PCA coordinates suitable for interactive Emperor visualization.

## When to use

Use when you have a file-by-chemical annotation table (GNPS spectral library matches per MassIVE file) and want to reproduce ReDU's repository-scale, unsupervised PCA ordination of public tandem MS files.

## When NOT to use

- Input is already a PC-coordinate table or an existing Emperor mapping file — go straight to visualization instead of recomputing PCA
- The goal is supervised or hypothesis-driven comparison of cohorts — use ReDU's Group Comparator or metadata-filtered molecular networking rather than unsupervised PCA
- Data have not been chemically annotated (no GNPS library-search results per file) — there is no chemical-annotation feature table to ordinate

## Inputs

- Repository-wide chemical annotation matrix (MassIVE file x annotated chemical, from GNPS MS2 spectral library search, e.g. ReDU database/global_tasks.tsv batch dump)
- ReDU identification database file set (MSV000084206) for restricting features/files
- ReDU sample information (metadata) for coloring/filtering points in Emperor

## Outputs

- Per-file PCA coordinate table (e.g. PCA_coordinates.tsv, one row per analyzed file with principal-component coordinate columns)
- Emperor-ready PCA visualization (interactive 3D plot with selectable PC axes and metadata coloring)

## How to apply

Load the repository-wide matrix mapping each MassIVE file to the chemicals annotated in that file by GNPS spectral library search (MS2 product ion spectra matched against reference MS2 fragmentation patterns). Restrict the feature table to the annotated chemical features present across all files in the ReDU identification database (MSV000084206). Run unsupervised PCA (e.g. scikit-learn PCA) on the chemical-annotation feature table to derive principal component scores per file — ReDU uses PCA precisely because it analyzes the results in an unsupervised manner. Export the per-file PC coordinate table (e.g. PCA_coordinates.tsv, one row per analyzed file) for upload into Emperor, where users can change PC axes, color points by sample information (metadata from the ReDU File Selector), and filter plotted data points. Validation: the TSV parses, contains one row per analyzed file with PC coordinate columns, and reproduces the PCA displayed in ReDU's Emperor views.

## Related tools

- **ReDU** (Source of the per-file chemical annotation matrix and sample information; provides the identification database (MSV000084206) whose files define the analysis universe) — https://github.com/mwang87/ReDU-MS2-GNPS
- **GNPS** (Performs the MS2 spectral library search that produces the chemical annotations (feature table) ordinated by PCA) — https://github.com/CCMS-UCSD/GNPSDocumentation
- **Emperor** (Interactive visualization of the PCA results: change PC axes, color points by sample information, and filter plotted data points) — https://github.com/biocore/emperor
- **MassIVE** (Public repository whose file identifiers (MassIVE IDs) index rows of the annotation matrix)

## Examples

```
python -c "import pandas as pd; from sklearn.decomposition import PCA; X=pd.read_csv('matrix.tsv',sep='\t',index_col=0); pcs=PCA().fit_transform(X); pcs.to_csv('PCA_coordinates.tsv',sep='\t')"
```

## Evaluation signals

- Output TSV parses and contains one row per analyzed MassIVE file, each with principal-component coordinate columns
- Ordination is unsupervised: no metadata labels are used in fitting the PCA, only the chemical-annotation feature table
- Feature table restricted to chemical features annotated across the files of the ReDU identification database (MSV000084206)
- Coordinates load into Emperor and reproduce the same relative placement of files as ReDU's global PCA view (up to sign/rotation of components)
- Point coloring/filtering in Emperor reflects ReDU sample-information (metadata) attributes rather than PCA inputs

## Limitations

- The same chemical can have multiple GNPS annotations due to slight variation in MS2 spectra (m/z or abundance), inflating feature-table columns
- Recently contributed data are not displayed immediately — ReDU data are updated periodically, so a fresh reproduction may lag the live site
- GNPS spectral library annotations are level 2 or level 3 (putative) by the 2007 metabolomics standards initiative; PCA reflects putative annotations only
- After a database rebuild, ReDU requires a full autopopulate before global PCA views are valid

## Evidence

- [abstract] PCA of chemical annotations and visualize with Emperor: "Principal components analysis (PCA) was used to analyze the results in an unsupervised manner. The results are plotted using Emperor"
- [abstract] Emperor visualization features: "Emperor is a visualization tool used to display the results of principal component analysis (PCA) performed on the chemicals annotated in each file in ReDU"
- [abstract] GNPS annotation mechanism: [evidence span withheld — anchor status: not_found]
- [abstract] Duplicate annotations caveat: "The same chemical can have multiple GNPS annotations. Slight variation in the MS2 spectra (m/z or abundance) cause the pattern to match different reference MS2 spectra"
- [readme] ReDU update procedure: [evidence span withheld — anchor status: external_doc]
