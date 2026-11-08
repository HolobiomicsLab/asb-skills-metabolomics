---
name: leiden-community-detection-on-cell-graphs
description: Use when you have a per-cell mean marker-intensity matrix from multiplexed
  IMC protein imaging and need single-cell cluster/cell-type assignments across multiple
  imaging regions or patients.
when_to_use_negative:
- Input is raw IMC image stacks rather than a segmented per-cell intensity feature
  table — segmentation (scSpaMet notebook 03/04) must come first
- Data are metabolite/SIMS pixel-level measurements rather than per-cell protein marker
  intensities — this skill annotates protein-derived cell types
- Cells come from a single homogeneous region where batch correction and multi-region
  clustering add no value
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scSpaMet
  role: Provides the segmented single-cell protein intensity extraction (notebook
    04, single-cell proteomics clustering) that feeds the Leiden graph
  repo: https://github.com/coskunlab/scSpaMet
- name: Scanorama
  role: Batch-corrects the z-score normalized single-cell protein matrices across
    imaging regions/patients before graph construction
- name: Leiden (scanpy.tl.leiden)
  role: Community detection on the kNN graph of Scanorama-corrected cells to assign
    cluster labels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:ccaa7f0bcc58b34c26e1117b7b56dc6441bdb12b63de4e11b233d6424e1312ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coskunlab/scSpaMet
  tools:
  - Leiden
  - Leiden (scanpy.tl.leiden)
  - Scanorama
  - scSpaMet
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# leiden-community-detection-on-cell-graphs

## Summary

Assigns every segmented IMC single cell an unsupervised cell-type cluster label by running Leiden community detection on a kNN graph built over normalized (and Scanorama batch-corrected) per-cell protein marker intensities from the scSpaMet pipeline.

## When to use

Use when you have a per-cell mean marker-intensity matrix from multiplexed IMC protein imaging and need single-cell cluster/cell-type assignments across multiple imaging regions or patients.

## When NOT to use

- Input is raw IMC image stacks rather than a segmented per-cell intensity feature table — segmentation (scSpaMet notebook 03/04) must come first
- Data are metabolite/SIMS pixel-level measurements rather than per-cell protein marker intensities — this skill annotates protein-derived cell types
- Cells come from a single homogeneous region where batch correction and multi-region clustering add no value

## Inputs

- Per-cell mean marker-intensity feature matrix from IMC single-cell segmentation (pandas DataFrame)
- Batch/region/patient index per cell (for Scanorama correction)
- Processed single-cell matrices from Zenodo deposit 10.5281/zenodo.6784251

## Outputs

- cell_cluster_labels.csv with one row per segmented cell per tissue: cell barcode/ID and assigned Leiden cluster label

## How to apply

Load the per-cell mean marker-intensity feature matrix produced by scSpaMet single-cell segmentation (processed matrices from Zenodo deposit 10.5281/zenodo.6784251) into a pandas DataFrame, then z-score normalize each of the ~25 protein markers so every marker has zero mean and unit variance across cells. Batch-correct the normalized cells across imaging regions/patients with Scanorama (scanorama.correct on the batch-indexed list of cell matrices). Build a k-nearest-neighbor graph on the Scanorama-corrected embedding and run Leiden community detection (scanpy.tl.leiden) so each cell receives exactly one community label. Write the cell barcode/ID and Leiden cluster label per tissue to cell_cluster_labels.csv; the paper reports no numeric quality threshold, so validation is a qualitative completeness check that every segmented cell carries one non-missing label.

## Related tools

- **scSpaMet** (Provides the segmented single-cell protein intensity extraction (notebook 04, single-cell proteomics clustering) that feeds the Leiden graph) — https://github.com/coskunlab/scSpaMet
- **Scanorama** (Batch-corrects the z-score normalized single-cell protein matrices across imaging regions/patients before graph construction)
- **Leiden (scanpy.tl.leiden)** (Community detection on the kNN graph of Scanorama-corrected cells to assign cluster labels)

## Examples

```
import scanpy as sc, scanorama; corrected = scanorama.correct([batch_a, batch_b]); adata = sc.AnnData(np.vstack(corrected[0])); sc.pp.neighbors(adata); sc.tl.leiden(adata, key_added='leiden')
```

## Evaluation signals

- Output table cell_cluster_labels.csv exists and is readable
- Table has one row per segmented cell per tissue (no duplicated or dropped cell IDs)
- Every cell carries exactly one Leiden cluster label with no missing/null assignments
- Z-score normalization leaves each marker column with mean 0 and unit variance before batch correction
- Cluster labels are consistent across Scanorama-corrected batches (cells from one tissue region appear in one labeling pass)

## Limitations

- The paper reports no numeric clustering-quality threshold (e.g. silhouette or modularity cutoff); acceptance is a qualitative completeness check only
- Leiden annotation is unsupervised — cluster identity must still be manually mapped to cell types using marker expression
- Batch correction quality depends on having per-batch cell matrices from imaging regions/patients; single-batch data gain little from Scanorama

## Evidence

- [readme] readme: "04 Single-cell level intensity extraction and single-cell proteomics clustering"
- [results] results: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
- [abstract] abstract: "The scSpaMet analysis revealed cell type-dependent metabolite profiles and local metabolite competition of neighboring single cells in human tissues"
- [intro] intro: "Several algorithms have been developed for the integration of multi-omics data such as MNN19, Scanorama20, Conos21, MARIO22, Seurat23, LIGER24, and SEAM14"
