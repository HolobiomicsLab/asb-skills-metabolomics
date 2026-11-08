---
name: single-cell-proteomics-clustering
description: Use when you have a per-cell mean marker-intensity feature matrix from
  single-cell segmentation of multiplexed IMC protein images (multiple imaging regions
  or patients) and need to label every segmented cell with a cell-type cluster.
when_to_use_negative:
- Input is raw IMC image data rather than a segmented per-cell feature matrix — segmentation
  (scSpaMet notebook 03 or Mesmer) must precede clustering
- The goal is joint protein-metabolite embedding of metabolite states rather than
  cell-type assignment from protein markers alone — use the scSpaMet VAE joint embedding
  (notebook 05) instead
- Spatial metabolomic data lacking shared protein markers and identified cell types,
  which the paper notes are unsuitable for standard multi-omics integration approaches
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: scSpaMet
  role: Pipeline providing single-cell segmentation, intensity extraction, and the
    single-cell proteomics clustering notebook (04) this skill reproduces
  repo: https://github.com/coskunlab/scSpaMet
- name: Scanorama
  role: Batch correction of z-score-normalized marker-intensity matrices across imaging
    regions/patients before kNN graph construction
- name: Leiden
  role: Community detection on the Scanorama-corrected kNN embedding (scanpy.tl.leiden)
    to assign one cluster label per cell
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:c44db1f5fefae31d8ed22ee7233d62b7fdbab916beeb277704b50cb2ad136716
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coskunlab/scSpaMet
  tools:
  - Leiden
  - Scanorama
  - scSpaMet
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-proteomics-clustering

## Summary

Assigns single cells from multiplexed IMC protein imaging to unsupervised cell-type clusters by z-score normalizing the per-cell mean marker-intensity matrix, batch-correcting with Scanorama, and running Leiden community detection on a kNN graph, as implemented in the scSpaMet pipeline for lung cancer, tonsil, and endometrium tissues.

## When to use

Use when you have a per-cell mean marker-intensity feature matrix from single-cell segmentation of multiplexed IMC protein images (multiple imaging regions or patients) and need to label every segmented cell with a cell-type cluster.

## When NOT to use

- Input is raw IMC image data rather than a segmented per-cell feature matrix — segmentation (scSpaMet notebook 03 or Mesmer) must precede clustering
- The goal is joint protein-metabolite embedding of metabolite states rather than cell-type assignment from protein markers alone — use the scSpaMet VAE joint embedding (notebook 05) instead
- Spatial metabolomic data lacking shared protein markers and identified cell types, which the paper notes are unsuitable for standard multi-omics integration approaches

## Inputs

- Per-cell mean marker-intensity feature matrix (one row per segmented cell, one column per protein marker, ~25 protein markers) produced by single-cell segmentation of multiplexed IMC images
- Batch identifiers mapping each cell to its imaging region or patient

## Outputs

- cell_cluster_labels.csv: cell barcode/ID and assigned Leiden cluster label for each tissue, one row per segmented cell per tissue
- Unsupervised cell-type cluster assignments annotated as cell types

## How to apply

Load the per-cell mean marker-intensity feature matrix (processed matrices from the ScSpaMet Zenodo deposit 10.5281/zenodo.6784251) into a pandas DataFrame, then apply z-score normalization per protein marker (scipy.stats.zscore or sklearn StandardScaler) so each marker has zero mean and unit variance across cells. Batch-correct the normalized matrix across imaging regions/patients with scanorama.correct using the batch-indexed list of cell matrices, since spatial metabolomic/proteomic data from different regions require multi-omics integration. Build a k-nearest-neighbor graph on the Scanorama-corrected embedding and run Leiden community detection (scanpy.tl.leiden) to assign a cluster label to every cell. The paper applies this within scSpaMet's notebook 04 (single-cell proteomics clustering) to >25 protein markers per cell, yielding 19507, 31156, and 8215 annotated single cells in lung cancer, tonsil, and endometrium tissues respectively.

## Related tools

- **scSpaMet** (Pipeline providing single-cell segmentation, intensity extraction, and the single-cell proteomics clustering notebook (04) this skill reproduces) — https://github.com/coskunlab/scSpaMet
- **Scanorama** (Batch correction of z-score-normalized marker-intensity matrices across imaging regions/patients before kNN graph construction)
- **Leiden** (Community detection on the Scanorama-corrected kNN embedding (scanpy.tl.leiden) to assign one cluster label per cell)

## Examples

```
import scanpy as sc; sc.pp.neighbors(adata, use_rep='X_scanorama'); sc.tl.leiden(adata)
```

## Evaluation signals

- Output table cell_cluster_labels.csv exists and has exactly one row per segmented cell per tissue
- Every cell carries exactly one Leiden cluster label with no missing assignments (qualitative completeness check — the paper reports no numeric clustering-quality threshold)
- Per-marker z-score normalization confirmed: each marker column has zero mean and unit variance before batch correction
- Cluster labels are present for all three tissues profiled (e.g., totals matching 19507 lung cancer, 31156 tonsil, 8215 endometrium single cells)
- Batch correction applied across all imaging regions/patients so no batch dominates the kNN graph structure

## Limitations

- No numeric clustering-quality threshold is reported; acceptance is a qualitative completeness check that every segmented cell receives exactly one cluster label
- MSI-derived spatial data are 'unsuitable for integration with other modalities because of their untargeted nature, lack of shared markers, and identified cell types', limiting extension of this protein-marker clustering logic to metabolite channels
- Cluster biological annotation (cell-type naming) is manual and relies on marker knowledge; the Leiden labels themselves are unsupervised

## Evidence

- [other] z-score normalization per marker: [evidence span withheld — anchor status: not_found]
- [other] kNN graph + Leiden labeling: [evidence span withheld — anchor status: not_found]
- [intro] integration algorithm context: "Several algorithms have been developed for the integration of multi-omics data such as MNN19, Scanorama20, Conos21"
- [readme] pipeline notebook: "04 Single-cell level intensity extraction and single-cell proteomics clustering"
- [abstract] cells profiled per tissue: "profile cell types and spatial metabolomic maps of 19507, 31156, and 8215 single cells in human lung cancer, tonsil, and endometrium tissues, respectively"
