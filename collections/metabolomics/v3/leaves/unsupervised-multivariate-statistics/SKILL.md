---
name: unsupervised-multivariate-statistics
description: Use when you have a file-by-chemical annotation matrix built from GNPS
  MS2 spectral library matches and want an unsupervised, label-free view of how public
  tandem MS files group by their annotated chemistry.
when_to_use_negative:
- You need supervised or group-comparison statistics (e.g. enrichment between metadata-defined
  groups) — use ReDU's Group Comparator / Chemical Explorer instead of PCA
- Your input is raw MS2 spectra rather than a processed GNPS annotation matrix — chemical
  annotation via GNPS spectral library search must precede PCA
- The file collection exceeds GNPS recommended limits (5000 files for library search),
  since annotation scale drives the PCA input
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: performs the chemical annotation (MS2 spectral library search) that produces
    the feature matrix PCA runs on
  repo: https://github.com/mwang87/ReDU-MS2-GNPS
- name: ReDU
  role: aggregates the per-file chemical annotations across MassIVE public data and
    serves the global PCA
  repo: https://github.com/mwang87/ReDU-MS2-GNPS
- name: Emperor
  role: interactive visualization of the PCA results (PC axes, coloring, filtering)
  repo: https://github.com/biocore/emperor
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-020-0916-7
    title: 'ReDU: a framework to find and reanalyze public mass spectrometry data'
  source_content_hash: sha256:041b37fec955fbd62dde23555bb390918dd94691632881816b222c8e25279d8b
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
  - ReDU
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# unsupervised-multivariate-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply unsupervised PCA to the repository-scale chemical-annotation matrix of ReDU (GNPS spectral library annotations per MassIVE file) to derive per-file principal component scores, and expose them for interactive visualization in Emperor.

## When to use

Use when you have a file-by-chemical annotation matrix built from GNPS MS2 spectral library matches and want an unsupervised, label-free view of how public tandem MS files group by their annotated chemistry.

## When NOT to use

- You need supervised or group-comparison statistics (e.g. enrichment between metadata-defined groups) — use ReDU's Group Comparator / Chemical Explorer instead of PCA
- Your input is raw MS2 spectra rather than a processed GNPS annotation matrix — chemical annotation via GNPS spectral library search must precede PCA
- The file collection exceeds GNPS recommended limits (5000 files for library search), since annotation scale drives the PCA input

## Inputs

- Chemical-annotation feature table mapping each MassIVE file (from MSV000084206) to GNPS spectral library annotations of MS2 product ion scans
- ReDU identification database of public tandem MS files (MassIVE ID MSV000084206)

## Outputs

- Per-file principal component coordinate table (e.g. PCA_coordinates.tsv) with one row per analyzed file and PC score columns
- Interactive Emperor visualization (PCA score plot) supporting PC axis switching, coloring by sample information, and plotted-data filtering

## How to apply

Load the repository-wide matrix mapping each MassIVE file (MSV000084206 is the ReDU identification database) to the chemicals annotated in that file by GNPS spectral library search of MS2 product ion spectra. Restrict the matrix to the annotated chemical features across all files, treating each annotation level 2/3 spectral match as a binary feature (note the same chemical can carry multiple GNPS annotations due to slight m/z or abundance variation in MS2 spectra). Fit an unsupervised PCA (e.g. scikit-learn) on this feature table with no metadata labels, deriving principal component scores per file. Export the per-file PC coordinates as a tabular score file (e.g. PCA_coordinates.tsv) with one row per analyzed file, formatted for upload into Emperor. ReDU itself plots these results in Emperor, where users change PC axes, color points by sample information, and filter plotted data points.

## Related tools

- **GNPS** (performs the chemical annotation (MS2 spectral library search) that produces the feature matrix PCA runs on) — https://github.com/mwang87/ReDU-MS2-GNPS
- **ReDU** (aggregates the per-file chemical annotations across MassIVE public data and serves the global PCA) — https://github.com/mwang87/ReDU-MS2-GNPS
- **Emperor** (interactive visualization of the PCA results (PC axes, coloring, filtering)) — https://github.com/biocore/emperor

## Examples

```
from sklearn.decomposition import PCA; import pandas as pd; coords = PCA(n_components=3).fit_transform(pd.read_csv('annotations_matrix.tsv', sep='\t')); pd.DataFrame(coords, index=feature_table.index).to_csv('PCA_coordinates.tsv', sep='\t')
```

## Evaluation signals

- Output TSV parses and contains one row per analyzed MassIVE file with principal-component coordinate columns
- PCA is fit without any metadata labels (unsupervised) — group/sample-information variables must not enter the fit
- Coordinate table is directly loadable into Emperor and reproduces the PCA ReDU displays in its global PCA view
- When rebuilding after a database update, ReDU's own procedure requires waiting for full autopopulation of identifications before the global PCA views are computed
- Number of analyzed files matches the file count reported in the ReDU File Selector for MSV000084206

## Limitations

- The same chemical can have multiple GNPS annotations because slight variation in MS2 spectra (m/z or abundance) matches different reference spectra; annotations are only level 2/3 confidence
- ReDU data are updated periodically and will not display recently contributed data immediately
- ReDU only interacts with MassIVE; data from other repositories are not included
- GNPS recommended file limits: 3000 for molecular networking and 5000 for library search

## Evidence

- [abstract] Chemical annotation is performed in GNPS by comparing MS2 spectra, specifically product ion spectra, with reference MS2 fragmentation patterns: [evidence span withheld — anchor status: not_found]
- [readme] Remove all untracked files in temp, this will be for the global pca: [evidence span withheld — anchor status: external_doc]
