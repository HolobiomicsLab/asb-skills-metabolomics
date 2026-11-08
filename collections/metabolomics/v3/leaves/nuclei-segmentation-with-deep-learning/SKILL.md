---
name: nuclei-segmentation-with-deep-learning
description: Use when processing multiplexed IMC (or similar multiplexed protein imaging)
  tissue stacks where individual cells must be segmented to extract per-cell protein/metabolite
  intensities and spatial locations, e.g., lung cancer, tonsil, or endometrium tissues.
when_to_use_negative:
- The input is already a single-cell feature/intensity table with per-cell protein
  and metabolite values — segmentation is unnecessary; proceed to clustering or joint
  embedding.
- Images are TOF-SIMS/SIMS metabolite-only acquisitions without a corresponding IMC
  nuclear channel — Cellpose nuclei segmentation on Histone 3/intercalator cannot
  be run; instead register or pixel-cluster the SIMS data.
- The goal is tissue-level or pixel-level metabolite mapping rather than single-cell
  extraction — pixel clustering of SIMS data is the appropriate route.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0621
tools:
- name: scSpaMet
  role: Framework wrapping Cellpose nuclei segmentation and Mesmer whole-cell segmentation;
    notebooks 03 (segmentation and mask visualization) and 11 (Mesmer pipeline) implement
    this skill
  repo: https://github.com/coskunlab/scSpaMet
- name: Mesmer
  role: Whole-cell segmentation pipeline applied via the scSpaMet wrapper to expand
    nuclei into cytosol/whole-cell masks using membrane and cytosolic protein markers
  repo: https://github.com/coskunlab/scSpaMet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:806b2b1cf2c62dd48212c90efc1fd42159e0217a43ed9537cec93f6321bbace7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coskunlab/scSpaMet
  tools:
  - Mesmer
  - scSpaMet
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# nuclei-segmentation-with-deep-learning

## Summary

Deep-learning-based single-cell segmentation of multiplexed IMC tissue images, using Cellpose on nuclear channels (DNA-intercalator/Histone 3) followed by Mesmer whole-cell mask expansion, as implemented in the scSpaMet framework to recover per-cell protein and metabolite measurements.

## When to use

Use when processing multiplexed IMC (or similar multiplexed protein imaging) tissue stacks where individual cells must be segmented to extract per-cell protein/metabolite intensities and spatial locations, e.g., lung cancer, tonsil, or endometrium tissues.

## When NOT to use

- The input is already a single-cell feature/intensity table with per-cell protein and metabolite values — segmentation is unnecessary; proceed to clustering or joint embedding.
- Images are TOF-SIMS/SIMS metabolite-only acquisitions without a corresponding IMC nuclear channel — Cellpose nuclei segmentation on Histone 3/intercalator cannot be run; instead register or pixel-cluster the SIMS data.
- The goal is tissue-level or pixel-level metabolite mapping rather than single-cell extraction — pixel clustering of SIMS data is the appropriate route.

## Inputs

- Raw multiplexed IMC image stacks for lung cancer TMA (TissueArray BS04081a), healthy human tonsil, and human endometrium tissues (Zenodo doi:10.5281/zenodo.6784251)
- DNA-intercalator / Histone 3 nuclear channel images
- Membrane and cytosolic protein marker channels (from metal-isotope conjugated antibody staining)

## Outputs

- Per-imaging-region nuclear segmentation masks (Cellpose)
- Whole-cell/cytosol segmentation masks (Mesmer)
- Segmentation mask visualizations per region
- Per-tissue single-cell counts (e.g., cell_counts.csv): 19507 lung / 31156 tonsil / 8215 endometrium

## How to apply

Load the multiplexed IMC image stacks from the Zenodo deposit (doi:10.5281/zenodo.6784251) and run Cellpose nuclei segmentation on the DNA-intercalator/Histone 3 nuclear channel for each imaging region, as in scSpaMet notebook 03. Then apply the Mesmer pipeline via the scSpaMet wrapper (notebook 11) to expand nuclei into whole-cell/cytosol segmentation masks using membrane and cytosolic protein markers. Visually inspect the resulting masks per imaging region to verify nuclear and whole-cell boundary quality before downstream intensity extraction. Finally, count segmented single cells per tissue and aggregate, expecting on the order of 19507 (lung cancer), 31156 (tonsil), and 8215 (endometrium) cells as reported by scSpaMet.

## Related tools

- **scSpaMet** (Framework wrapping Cellpose nuclei segmentation and Mesmer whole-cell segmentation; notebooks 03 (segmentation and mask visualization) and 11 (Mesmer pipeline) implement this skill) — https://github.com/coskunlab/scSpaMet
- **Mesmer** (Whole-cell segmentation pipeline applied via the scSpaMet wrapper to expand nuclei into cytosol/whole-cell masks using membrane and cytosolic protein markers) — https://github.com/coskunlab/scSpaMet

## Examples

```
jupyter notebook notebooks/03_segmentation.ipynb  # Cellpose nuclei segmentation on Histone 3 channel, then notebooks/11 Mesmer whole-cell segmentation
```

## Evaluation signals

- Per-tissue cell counts reproduce the reported numbers within a reasonable range: 19507 (lung cancer), 31156 (tonsil), 8215 (endometrium).
- Segmentation masks visually align with nuclear (Histone 3/DNA-intercalator) and whole-cell boundaries when visualized per imaging region.
- Segmented cell counts per imaging region aggregate consistently to tissue-level counts without orphaned or duplicated masks.
- Downstream single-cell protein and metabolite intensities can be extracted for every segmented cell with valid spatial locations.

## Limitations

- Segmentation quality depends on the nuclear channel (Histone 3/DNA-intercalator) signal; crowded tissue regions may cause boundary errors requiring visual verification.
- Whole-cell masks rely on membrane/cytosolic marker quality from the IMC panel (25 protein markers), so poorly stained regions yield unreliable cytosol expansion.
- Cross-modality single-cell registration with the TOF-SIMS (Phosphate 79 m/z) channel must be correct before metabolite intensities can be attached to segmented cells; registration errors propagate into per-cell measurements.

## Evidence

- [results] Single-cell segmentation to extract protein and metabolite expression levels and spatial locations: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
- [abstract] Cell counts abstract: "profile cell types and spatial metabolomic maps of 19507, 31156, and 8215 single cells in human lung cancer, tonsil, and endometrium tissues, respectively"
- [readme] README notebooks: [evidence span withheld — anchor status: not_found]
- [readme] Raw data Zenodo: "You can find the raw data here: https://doi.org/10.5281/zenodo.6784251."
- [results] Cross-modality registration: "a cross-modality single-cell registration pipeline was developed utilizing Histone 3 and Intercalator markers in the IMC dataset, and Phosphate 79 m/z channels in TOF-SIMS dataset"
