---
name: scanorama-multi-omics-batch-correction
description: Use when assigning single-cell cluster/cell-type labels to per-cell IMC
  protein intensity tables spanning multiple imaging regions or patients, where batch
  effects must be removed before Leiden clustering.
when_to_use_negative:
- Input is raw pixel-level SIMS/metabolite data rather than a per-cell protein feature
  matrix — scSpaMet notes spatial metabolomic data are unsuitable for integration
  because of their untargeted nature and lack of shared markers
- Data come from a single imaging region/patient with no batch structure to correct
- The goal is joint protein-metabolite embedding (VAE-based in notebook 05), not protein-only
  cell-type clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Scanorama
  role: Batch-corrects the batch-indexed, z-scored per-cell IMC protein matrices via
    scanorama.correct before clustering
- name: scSpaMet
  role: Source pipeline providing the per-cell segmentation-derived intensity matrices
    and the single-cell proteomics clustering notebook
  repo: github.com/coskunlab/scSpaMet
- name: Leiden (scanpy.tl.leiden)
  role: Leiden community detection on the kNN graph of the Scanorama-corrected embedding
    assigns the final cluster label to each cell
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:eb32436d78d849aebee6605e731c22b7b396322247ebe4bacfe819a0c6c68888
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Leiden (scanpy.tl.leiden)
  - Scanorama
  - scSpaMet
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scanorama-multi-omics-batch-correction

## Summary

Batch-correct z-scored per-cell protein marker intensities from multiplexed IMC imaging across imaging regions/patients using Scanorama, then cluster cells into cell types via Leiden on a kNN graph, as done in the scSpaMet pipeline (Nature Communications, 2023).

## When to use

Use when assigning single-cell cluster/cell-type labels to per-cell IMC protein intensity tables spanning multiple imaging regions or patients, where batch effects must be removed before Leiden clustering.

## When NOT to use

- Input is raw pixel-level SIMS/metabolite data rather than a per-cell protein feature matrix — scSpaMet notes spatial metabolomic data are unsuitable for integration because of their untargeted nature and lack of shared markers
- Data come from a single imaging region/patient with no batch structure to correct
- The goal is joint protein-metabolite embedding (VAE-based in notebook 05), not protein-only cell-type clustering

## Inputs

- Per-cell mean marker-intensity feature matrix from single-cell IMC segmentation (processed matrices from Zenodo 10.5281/zenodo.6784251)
- Batch identifiers mapping cells to imaging regions or patients (lung cancer, tonsil, endometrium tissues)
- List of shared protein markers (e.g. the 25 protein markers profiled by scSpaMet) common across batches

## Outputs

- cell_cluster_labels.csv: one row per segmented cell per tissue with cell barcode/ID and a single assigned Leiden cluster label
- Scanorama-corrected cell embedding in shared marker space

## How to apply

Load the per-cell mean marker-intensity feature matrix from single-cell segmentation (processed matrices from the scSpaMet Zenodo deposit, doi 10.5281/zenodo.6784251) into pandas DataFrames, one per batch (imaging region/patient). Apply z-score normalization per protein marker (scipy.stats.zscore or sklearn StandardScaler) so each of the ~25 protein markers has zero mean and unit variance across cells. Pass the batch-indexed list of cell matrices to scanorama.correct, which aligns batches by matching mutually nearest neighbors in shared marker space. Build a k-nearest-neighbor graph on the Scanorama-corrected embedding (scanpy.pp.neighbors) and run Leiden community detection (scanpy.tl.leiden) so every segmented cell receives exactly one cluster label. The paper reports no numeric clustering-quality threshold, so validation is a qualitative completeness check on the emitted cell_cluster_labels.csv.

## Related tools

- **Scanorama** (Batch-corrects the batch-indexed, z-scored per-cell IMC protein matrices via scanorama.correct before clustering)
- **scSpaMet** (Source pipeline providing the per-cell segmentation-derived intensity matrices and the single-cell proteomics clustering notebook) — github.com/coskunlab/scSpaMet
- **Leiden (scanpy.tl.leiden)** (Leiden community detection on the kNN graph of the Scanorama-corrected embedding assigns the final cluster label to each cell)

## Examples

```
corrected = scanorama.correct([X1, X2], genes=marker_list); sc.pp.neighbors(adata, use_rep='X_scanorama'); sc.tl.leiden(adata)
```

## Evaluation signals

- cell_cluster_labels.csv exists with exactly one row per segmented cell per tissue (19507 lung cancer, 31156 tonsil, 8215 endometrium cells in the paper)
- Every cell carries exactly one non-missing Leiden cluster label — the qualitative completeness acceptance criterion, as the paper reports no numeric clustering-quality threshold
- Per-marker z-scores have zero mean and unit variance across cells before batch correction
- All batches share the same marker dimensionality and gene/marker order, as required by scanorama.correct
- Post-correction, kNN graph connectivity and Leiden label assignment complete without error on every batch

## Limitations

- Spatial metabolomic (SIMS) data are not amenable to this correction path: scSpaMet notes they are 'unsuitable for integration with other modalities because of their untargeted nature, lack of shared markers, and identified cell types'
- No numeric clustering-quality threshold is reported in the paper; acceptance relies on qualitative completeness of assignments only
- TOF-SIMS-derived features suffer from challenges in metabolite annotation and coverage for lipids, amino acids, and metabolic pathways, limiting cross-modal feature sharing

## Evidence

- [intro] intro: "Several algorithms have been developed for the integration of multi-omics data such as MNN19, Scanorama20, Conos21, MARIO22, Seurat23, LIGER24, and SEAM14"
- [readme] readme: "04 Single-cell level intensity extraction and single-cell proteomics clustering"
- [results] results: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
