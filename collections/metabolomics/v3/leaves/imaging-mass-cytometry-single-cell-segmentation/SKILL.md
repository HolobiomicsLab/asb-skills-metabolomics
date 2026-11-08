---
name: imaging-mass-cytometry-single-cell-segmentation
description: Use when raw multiplexed IMC image stacks must be converted into per-cell
  segmentation masks (nuclear and whole-cell) so that single-cell protein/metabolite
  intensities and spatial locations can be extracted and cell counts reported.
when_to_use_negative:
- Input is already a single-cell feature/expression table with no images to segment
- The task is cross-modality IMC-to-TOF-SIMS registration rather than IMC-only cell
  segmentation
- Data are non-IMC modalities (e.g., MALDI or raw TOF-SIMS ion images) lacking the
  nuclear channels Cellpose relies on
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3518
tools:
- name: scSpaMet
  role: Framework providing the image processing utilities, segmentation/visualization
    notebooks, and Mesmer wrapper used to segment single cells and extract intensities
  repo: github.com/coskunlab/ScSpaMet
- name: Mesmer
  role: Deep-learning whole-cell segmentation pipeline applied via the scSpaMet wrapper
    to expand nuclei masks into whole-cell/cytosol masks using membrane and cytosolic
    markers
  repo: github.com/coskunlab/scSpaMet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:19e6714d8c61f553000e80fa1e1da5e4d8e76c88cebed4c2866354617a3748e5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mesmer
  - scSpaMet
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# imaging-mass-cytometry-single-cell-segmentation

## Summary

Segment single cells in multiplexed imaging mass cytometry (IMC) images using Cellpose nuclei segmentation on DNA-intercalator/Histone 3 channels followed by Mesmer whole-cell mask expansion, enabling per-cell protein and metabolite extraction in the scSpaMet framework.

## When to use

Use when raw multiplexed IMC image stacks must be converted into per-cell segmentation masks (nuclear and whole-cell) so that single-cell protein/metabolite intensities and spatial locations can be extracted and cell counts reported.

## When NOT to use

- Input is already a single-cell feature/expression table with no images to segment
- The task is cross-modality IMC-to-TOF-SIMS registration rather than IMC-only cell segmentation
- Data are non-IMC modalities (e.g., MALDI or raw TOF-SIMS ion images) lacking the nuclear channels Cellpose relies on

## Inputs

- Raw multiplexed IMC image stacks from Zenodo doi:10.5281/zenodo.6784251 (lung adenocarcinoma grade III TMA BS04081a, healthy human tonsil, human endometrium)
- DNA-intercalator/Histone 3 nuclear channel images for nuclei segmentation
- Membrane and cytosolic protein marker channels for whole-cell mask expansion

## Outputs

- Nuclear segmentation masks per imaging region
- Whole-cell/cytosol segmentation masks per imaging region
- Per-tissue segmented single-cell count table (e.g., cell_counts.csv) reproducing 19507 / 31156 / 8215 cells

## How to apply

Load the multiplexed IMC image stacks (from the Zenodo deposit doi:10.5281/zenodo.6784251 for lung cancer TMA, healthy tonsil, and human endometrium) with the scSpaMet processing utilities. Run Cellpose nuclei segmentation on the DNA-intercalator/Histone 3 nuclear channel for each imaging region, then apply the Mesmer pipeline via the scSpaMet wrapper (notebook 11 in the repository) to expand nuclei into whole-cell/cytosol segmentation masks using membrane and cytosolic protein markers. Visualize the resulting masks per imaging region (notebook 03) to verify nuclear and whole-cell boundary quality before intensity extraction. Count segmented single cells per tissue and compare against the reported profiled counts of 19507 (lung cancer), 31156 (tonsil), and 8215 (endometrium).

## Related tools

- **scSpaMet** (Framework providing the image processing utilities, segmentation/visualization notebooks, and Mesmer wrapper used to segment single cells and extract intensities) — github.com/coskunlab/ScSpaMet
- **Mesmer** (Deep-learning whole-cell segmentation pipeline applied via the scSpaMet wrapper to expand nuclei masks into whole-cell/cytosol masks using membrane and cytosolic markers) — github.com/coskunlab/scSpaMet

## Examples

```
jupyter notebook notebooks/03_single_cell_segmentation.ipynb  # then notebooks/11 Segmentation of single-cell with Mesmer pipeline
```

## Evaluation signals

- Per-tissue cell counts fall in a reasonable range around the reported values: 19507 (lung cancer), 31156 (tonsil), 8215 (endometrium)
- Visual inspection of rendered segmentation masks per imaging region shows nuclei boundaries (Cellpose on DNA-intercalator/Histone 3) and whole-cell boundaries (Mesmer) that match tissue morphology
- Every nuclear mask expands to a corresponding whole-cell mask via the Mesmer step with no orphaned or overlapping masks
- Downstream per-cell intensity extraction succeeds only for cells with valid whole-cell masks, confirming mask validity

## Limitations

- Segmentation quality depends on nuclear channel quality (DNA-intercalator/Histone 3); poor staining or registration artifacts propagate into masks
- The paper's discussion notes relatively small imaging regions from tissue samples due to the TOF-SIMS trade-off between imaging speed and spatial resolution, limiting the number of cells segmented per field
- Mask boundary quality in spatially crowded tissues must be verified visually before trusting intensity extraction

## Evidence

- [results] Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
- [abstract] Cell counts finding: "profile cell types and spatial metabolomic maps of 19507, 31156, and 8215 single cells in human lung cancer, tonsil, and endometrium tissues, respectively"
- [readme] README notebook listing: "- 03 Single-cell level segmentation and visualization of segmentation masks"
- [readme] README Mesmer notebook: "- 11 Segmentation of single-cell with Mesmer pipeline"
