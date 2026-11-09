---
name: single-cell-protein-intensity-normalization
description: Use when you have a per-cell mean marker-intensity feature matrix from
  IMC single-cell segmentation spanning multiple imaging regions or patients, and
  you need comparable, normalized features for unsupervised cell-type clustering.
when_to_use_negative:
- Input is raw IMC image pixels or segmentation masks rather than an already-extracted
  per-cell intensity feature matrix — segmentation and intensity extraction must be
  done first (README notebook 03/04)
- The goal is joint protein-metabolite analysis (e.g. VAE joint embedding or trajectory
  inference), where a different scSpaMet analysis path is needed rather than protein-only
  normalization-plus-clustering
- Only one imaging region/patient is present, so Scanorama batch correction across
  batches has no batches to integrate
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: scSpaMet
  role: Pipeline providing the segmented single-cell protein intensity matrices and
    the single-cell proteomics clustering notebook (notebook 04) that this normalization
    feeds
  repo: github.com/coskunlab/ScSpaMet
- name: Scanorama
  role: Batch-corrects the z-score normalized marker-intensity matrix across imaging
    regions/patients before graph construction and Leiden clustering
- name: Leiden
  role: Community detection on the KNN graph built from the Scanorama-corrected embedding,
    assigning one cluster label to every cell
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:1330899adc0174d26b19a7f7569d40c1048b578376c5a1fe722840099364dda9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Leiden
  - Scanorama
  - scSpaMet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-protein-intensity-normalization

## Summary

Normalizes per-cell mean marker-intensity matrices from multiplexed IMC imaging data by z-scoring each protein marker and batch-correcting across imaging regions/patients with Scanorama, so that downstream Leiden clustering assigns comparable cell-type labels. It is a key step in the scSpaMet pipeline for assigning cell types to single cells from joint protein-metabolite profiling.

## When to use

Use when you have a per-cell mean marker-intensity feature matrix from IMC single-cell segmentation spanning multiple imaging regions or patients, and you need comparable, normalized features for unsupervised cell-type clustering.

## When NOT to use

- Input is raw IMC image pixels or segmentation masks rather than an already-extracted per-cell intensity feature matrix — segmentation and intensity extraction must be done first (README notebook 03/04)
- The goal is joint protein-metabolite analysis (e.g. VAE joint embedding or trajectory inference), where a different scSpaMet analysis path is needed rather than protein-only normalization-plus-clustering
- Only one imaging region/patient is present, so Scanorama batch correction across batches has no batches to integrate

## Inputs

- Per-cell mean marker-intensity feature matrix from IMC single-cell segmentation (one row per segmented cell, one column per protein marker), e.g. processed matrices from the ScSpaMet Zenodo deposit doi:10.5281/zenodo.6784251
- Batch index mapping each cell to its imaging region or patient

## Outputs

- cell_cluster_labels.csv containing the cell barcode/ID and assigned Leiden cluster label for each cell in each tissue
- Z-score normalized per-marker feature matrix (zero mean, unit variance per marker)
- Scanorama-corrected embedding across imaging regions/patients

## How to apply

Load the per-cell mean marker-intensity feature matrix produced by single-cell segmentation (processed matrices from the ScSpaMet Zenodo deposit 10.5281/zenodo.6784251) into a pandas DataFrame. Apply z-score normalization per protein marker with scipy.stats.zscore or sklearn StandardScaler so each marker has zero mean and unit variance across cells — this prevents high-abundance markers from dominating the neighbor graph. Batch-correct the normalized matrix across imaging regions/patients with scanorama.correct on the batch-indexed list of cell matrices. Build a k-nearest-neighbor graph on the Scanorama-corrected embedding and run Leiden community detection (scanpy.tl.leiden) to assign exactly one cluster label to every cell. The paper reports no numeric clustering-quality threshold, so the acceptance criterion is a qualitative completeness check: one row per segmented cell per tissue with no missing cluster assignments.

## Related tools

- **scSpaMet** (Pipeline providing the segmented single-cell protein intensity matrices and the single-cell proteomics clustering notebook (notebook 04) that this normalization feeds) — github.com/coskunlab/ScSpaMet
- **Scanorama** (Batch-corrects the z-score normalized marker-intensity matrix across imaging regions/patients before graph construction and Leiden clustering)
- **Leiden** (Community detection on the KNN graph built from the Scanorama-corrected embedding, assigning one cluster label to every cell)

## Examples

```
from sklearn.preprocessing import StandardScaler; Xs = StandardScaler().fit_transform(df); corrected = scanorama.correct([Xs[b] for b in batch_ids]); sc.tl.leiden(adata)
```

## Evaluation signals

- Output table cell_cluster_labels.csv exists with one row per segmented cell per tissue (e.g. the study profiled 19507 lung cancer, 31156 tonsil, and 8215 endometrium single cells)
- Every segmented cell carries exactly one Leiden cluster label with no missing assignments
- After z-score normalization, each protein marker column has zero mean and unit variance across cells
- Scanorama correction runs on a batch-indexed list of cell matrices without merging batches, and the KNN/Leiden graph is built on the corrected embedding
- Cluster assignments are produced for cells in all tissues/regions, matching the per-cell count of the input segmentation feature matrix

## Limitations

- The paper reports no numeric clustering-quality threshold, so validation relies on a qualitative completeness check rather than a quantitative metric
- Scanorama is one of several multi-omics/batch integration algorithms (MNN, Conos, MARIO, Seurat, LIGER, SEAM) — the choice is justified by the paper but alternative integrators could behave differently
- Normalization applies to the protein (IMC) modality only; spatial metabolomic data are unsuitable for this integration due to their untargeted nature and lack of shared markers

## Evidence

- [methods] z-score normalization per marker: [evidence span withheld — anchor status: not_found]
- [methods] Leiden on Scanorama-corrected embedding: [evidence span withheld — anchor status: not_found]
- [intro] Scanorama among integration algorithms: "Several algorithms have been developed for the integration of multi-omics data such as MNN19, Scanorama20, Conos21, MARIO22, Seurat23, LIGER24, and SEAM14"
- [results] single-cell segmentation source of intensities: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
- [readme] README notebook 04 covers clustering: "04 Single-cell level intensity extraction and single-cell proteomics clustering"
