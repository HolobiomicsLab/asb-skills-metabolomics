---
name: cross-tissue-cell-type-labeling
description: Use when you have a per-cell mean marker-intensity matrix from segmented
  multiplexed IMC (imaging mass cytometry) data spanning multiple imaging regions
  or patients and need unsupervised single-cell cluster labels that transfer across
  tissues.
when_to_use_negative:
- The cells are already annotated cell types (e.g. you only need metabolite-level
  comparisons within known cell types rather than new cluster labels)
- The task is metabolite/pixel-level analysis of TOF-SIMS SIMS data rather than protein-marker-based
  single-cell typing
- Input data are metabolite intensities without the 25-protein IMC panel — the article's
  cell-type tagging relies on metal-isotope conjugated antibody IMC imaging
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: scSpaMet
  role: Framework whose notebook 04 performs single-cell intensity extraction and
    single-cell proteomics clustering on the IMC data
  repo: github.com/coskunlab/ScSpaMet
- name: Scanorama
  role: Batch-corrects the z-score-normalized per-cell marker matrices across imaging
    regions/patients prior to clustering
- name: Leiden
  role: Community detection on the kNN graph of the corrected embedding assigns the
    final cluster label to each cell
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:5399d6ca15680e1336096ae36021a40ca072839e115bfe3295f1fecec1abc1b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Leiden
  - Scanorama
  - scSpaMet
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cross-tissue-cell-type-labeling

## Summary

Assigns a cell-type cluster label to every segmented cell from multiplexed IMC protein imaging by Leiden-clustering z-score-normalized, Scanorama-batch-corrected single-cell marker intensities, as done in the scSpaMet pipeline across lung cancer, tonsil, and endometrium tissues.

## When to use

Use when you have a per-cell mean marker-intensity matrix from segmented multiplexed IMC (imaging mass cytometry) data spanning multiple imaging regions or patients and need unsupervised single-cell cluster labels that transfer across tissues.

## When NOT to use

- The cells are already annotated cell types (e.g. you only need metabolite-level comparisons within known cell types rather than new cluster labels)
- The task is metabolite/pixel-level analysis of TOF-SIMS SIMS data rather than protein-marker-based single-cell typing
- Input data are metabolite intensities without the 25-protein IMC panel — the article's cell-type tagging relies on metal-isotope conjugated antibody IMC imaging

## Inputs

- Per-cell mean protein marker-intensity matrix from single-cell segmentation of multiplexed IMC images (one row per segmented cell per tissue region)
- Batch index / imaging-region or patient annotation mapping each cell to its source tissue

## Outputs

- cell_cluster_labels.csv containing the cell barcode/ID and assigned Leiden cluster label for each cell in each tissue
- Scanorama-corrected embedding used for the kNN graph

## How to apply

Load the per-cell mean marker-intensity matrix produced by single-cell segmentation (e.g. the processed matrices from the ScSpaMet Zenodo deposit 10.5281/zenodo.6784251) into a pandas DataFrame. Z-score normalize each protein marker (scipy.stats.zscore or sklearn StandardScaler) so markers are comparable across cells, then batch-correct the normalized matrices across imaging regions/patients with scanorama.correct, taking a batch-indexed list of cell matrices. Build a k-nearest-neighbor graph on the Scanorama-corrected embedding and run Leiden community detection (scanpy.tl.leiden) so every cell receives exactly one cluster label; the paper reports no numeric clustering-quality threshold, so completeness is the acceptance criterion. Rationale: IMC data span tissue regions from different patients (e.g. 19507 lung cancer, 31156 tonsil, and 8215 endometrium cells), so cross-modality/integration methods like Scanorama are needed before unsupervised clustering to prevent batch structure from dominating cell-type structure.

## Related tools

- **scSpaMet** (Framework whose notebook 04 performs single-cell intensity extraction and single-cell proteomics clustering on the IMC data) — github.com/coskunlab/ScSpaMet
- **Scanorama** (Batch-corrects the z-score-normalized per-cell marker matrices across imaging regions/patients prior to clustering)
- **Leiden** (Community detection on the kNN graph of the corrected embedding assigns the final cluster label to each cell)

## Examples

```
import scanpy as sc; corrected = scanorama.correct([X_region1, X_region2]); adata = sc.AnnData(np.vstack(corrected)); sc.pp.neighbors(adata); sc.tl.leiden(adata)
```

## Evaluation signals

- cell_cluster_labels.csv exists and has exactly one row per segmented cell per tissue (e.g. totals consistent with the reported 19507 lung cancer, 31156 tonsil, and 8215 endometrium cells)
- Every cell carries exactly one Leiden cluster label with no missing assignments (qualitative completeness check, since the paper reports no numeric clustering-quality threshold)
- Each protein marker column has zero mean and unit variance after z-score normalization before Scanorama correction
- The corrected embedding was built from batch-indexed per-region matrices (Scanorama was applied across patients/regions, not on a single pooled matrix)

## Limitations

- The paper reports no numeric clustering-quality threshold, so cluster validity must be judged by qualitative annotation and completeness rather than a cutoff metric
- Cluster labels are unsupervised and must be manually annotated as cell types; no ground-truth cell-type labels are provided by the pipeline

## Evidence

- [readme] readme notebook coverage: "04 Single-cell level intensity extraction and single-cell proteomics clustering"
- [intro] multi-omics integration algorithms cited: "Several algorithms have been developed for the integration of multi-omics data such as MNN19, Scanorama20, Conos21, MARIO22, Seurat23, LIGER24, and SEAM14"
- [abstract] scale of single-cell profiling across tissues: "profile cell types and spatial metabolomic maps of 19507, 31156, and 8215 single cells in human lung cancer, tonsil, and endometrium tissues, respectively"
- [results] segmentation basis for the feature matrix: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
