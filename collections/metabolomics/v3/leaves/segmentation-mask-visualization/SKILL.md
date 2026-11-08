---
name: segmentation-mask-visualization
description: Use when you have raw multiplexed IMC image stacks (e.g., DNA-intercalator/Histone
  3 nuclear channel) and have just run Cellpose/Mesmer segmentation, and need to verify
  nuclear and whole-cell boundary quality per imaging region.
when_to_use_negative:
- Input is already a single-cell expression/intensity table — visualization of masks
  adds no value at that stage
- Imaging data are unregistered between IMC and TOF-SIMS modalities and the goal is
  registration QA (use the notebook 02 registration-comparison workflow instead)
- Data are not multiplexed protein imaging (IMC) but, e.g., bulk metabolomics or non-spatial
  single-cell data lacking spatial context
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: scSpaMet
  role: Framework providing the notebooks (03 segmentation mask visualization, 11
    Mesmer segmentation) and plotting/io utilities (src/utils.py) used to render and
    inspect segmentation masks
  repo: github.com/coskunlab/ScSpaMet
- name: Mesmer
  role: Whole-cell/cytosol segmentation pipeline applied via the scSpaMet wrapper
    to expand Cellpose nuclei masks using membrane and cytosolic protein markers;
    masks produced are visualized
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:3c80201c8f619a627f6d7753a86e47f5b4ea774ec270da174759dee9a778a737
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mesmer
  - scSpaMet
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# segmentation-mask-visualization

## Summary

Visualize the nuclear and whole-cell segmentation masks generated for IMC imaging regions in the scSpaMet pipeline, so that boundary quality can be inspected before per-cell protein and metabolite intensity extraction and per-tissue cell counting.

## When to use

Use when you have raw multiplexed IMC image stacks (e.g., DNA-intercalator/Histone 3 nuclear channel) and have just run Cellpose/Mesmer segmentation, and need to verify nuclear and whole-cell boundary quality per imaging region.

## When NOT to use

- Input is already a single-cell expression/intensity table — visualization of masks adds no value at that stage
- Imaging data are unregistered between IMC and TOF-SIMS modalities and the goal is registration QA (use the notebook 02 registration-comparison workflow instead)
- Data are not multiplexed protein imaging (IMC) but, e.g., bulk metabolomics or non-spatial single-cell data lacking spatial context

## Inputs

- Multiplexed IMC image stacks per imaging region (DNA-intercalator/Histone 3 nuclear channel plus membrane/cytosolic protein marker channels)
- Cellpose nuclear segmentation masks
- Mesmer whole-cell/cytosol segmentation masks
- Raw data from Zenodo deposit doi:10.5281/zenodo.6784251 (lung cancer TMA TissueArray BS04081a, healthy tonsil, human endometrium)

## Outputs

- Per-imaging-region visualization of nuclear and whole-cell segmentation masks overlaid on IMC channels
- Validated segmentation masks suitable for downstream single-cell intensity extraction
- Per-tissue cell-count table (cell_counts.csv) with counts such as 19507 (lung cancer), 31156 (tonsil), 8215 (endometrium)

## How to apply

First run Cellpose nuclei segmentation on the DNA-intercalator/Histone 3 nuclear channel of each IMC imaging region, then apply the Mesmer pipeline via the scSpaMet wrapper to expand nuclei into whole-cell/cytosol masks using membrane and cytosolic protein markers. Render the resulting masks overlaid on the corresponding IMC channels for each imaging region using the notebook 03 workflow ('Single-cell level segmentation and visualization of segmentation masks'), using the plotting utilities in src/utils.py. Inspect whether nuclei and whole-cell boundaries align with visible cell structures in the lung cancer TMA, tonsil, and endometrium tissues. If boundary quality is poor, switch to or compare against the Mesmer-only segmentation path provided in notebook 11 before proceeding to intensity extraction (notebook 04). Only after visual verification should counts be aggregated per tissue and written to cell_counts.csv.

## Related tools

- **scSpaMet** (Framework providing the notebooks (03 segmentation mask visualization, 11 Mesmer segmentation) and plotting/io utilities (src/utils.py) used to render and inspect segmentation masks) — github.com/coskunlab/ScSpaMet
- **Mesmer** (Whole-cell/cytosol segmentation pipeline applied via the scSpaMet wrapper to expand Cellpose nuclei masks using membrane and cytosolic protein markers; masks produced are visualized) — github.com/coskunlab/ScSpaMet

## Evaluation signals

- Visual overlay shows nuclei masks contained within whole-cell/cytosol masks and boundaries aligned with visible cell structures in each imaging region
- Per-tissue counts aggregated after verified segmentation reproduce 19507 (lung cancer), 31156 (tonsil), and 8215 (endometrium) single cells within a reasonable range
- Every imaging region in the three tissue datasets has a rendered mask QA image (no region skipped between segmentation and intensity extraction)
- Mask objects are spatially registered with the corresponding IMC channels (Histone 3 / Intercalator markers) so extracted intensities map to real cell locations

## Limitations

- Segmentation quality depends on staining and tissue preservation: OCT-embedded frozen tissues show better metabolite preservation but less preserved tissue structures, which can impair boundaries
- TOF-SIMS imaging regions are relatively small due to the imaging-speed/spatial-resolution trade-off, limiting how much mask context can be inspected per region
- Mesmer-derived whole-cell masks and Cellpose nuclear masks may disagree; notebook 11 exists specifically to compare/segment with Mesmer as an alternative

## Evidence

- [readme] README notebook 03 supports the mask-visualization step: "03 Single-cell level segmentation and visualization of segmentation masks"
- [readme] README notebook 11 supports the Mesmer alternative path: "11 Segmentation of single-cell with Mesmer pipeline"
- [results] Article results section: purpose of segmentation downstream of visualization: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
- [abstract] Abstract: expected per-tissue single-cell counts to validate segmentation: "profile cell types and spatial metabolomic maps of 19507, 31156, and 8215 single cells in human lung cancer, tonsil, and endometrium tissues"
