---
name: whole-cell-cytosol-segmentation-mesmer
description: Use when IMC nuclear channel (DNA-intercalator/Histone 3) segmentation
  exists but whole-cell boundaries are needed to profile per-cell protein and metabolite
  signals in spatially crowded tissue.
when_to_use_negative:
- Input is already a per-cell feature/intensity table rather than raw multiplexed
  IMC images
- Data lack a nuclear channel (DNA-intercalator/Histone 3) or membrane/cytosolic markers,
  so neither Cellpose nuclei nor Mesmer whole-cell expansion can be seeded
- Goal is metabolite-only pixel clustering of SIMS data without single-cell resolution
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_2229
tools:
- name: scSpaMet
  role: Framework wrapping the segmentation workflow; notebooks 03 (mask visualization)
    and 11 (Mesmer segmentation) drive nuclei-to-whole-cell mask generation
  repo: github.com/coskunlab/ScSpaMet
- name: Mesmer
  role: Deep-learning segmentation pipeline used to expand Cellpose nuclear masks
    into whole-cell/cytosol masks from membrane and cytosolic protein markers
  repo: github.com/coskunlab/ScSpaMet
- name: Cellpose
  role: Nuclei segmentation on the DNA-intercalator/Histone 3 IMC channel, providing
    seeds for Mesmer whole-cell expansion
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:1bb3735ea6fda768ac16365dbb53e1f5fc58c550d244e919eb2835e8b7eb46d0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cellpose
  - Mesmer
  - scSpaMet
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# whole-cell-cytosol-segmentation-mesmer

## Summary

Expands Cellpose-derived nuclear masks into whole-cell/cytosol segmentation masks using the Mesmer pipeline via the scSpaMet wrapper, enabling per-single-cell extraction of protein (IMC) and metabolite (TOF-SIMS) intensities in crowded human tissues.

## When to use

Use when IMC nuclear channel (DNA-intercalator/Histone 3) segmentation exists but whole-cell boundaries are needed to profile per-cell protein and metabolite signals in spatially crowded tissue.

## When NOT to use

- Input is already a per-cell feature/intensity table rather than raw multiplexed IMC images
- Data lack a nuclear channel (DNA-intercalator/Histone 3) or membrane/cytosolic markers, so neither Cellpose nuclei nor Mesmer whole-cell expansion can be seeded
- Goal is metabolite-only pixel clustering of SIMS data without single-cell resolution

## Inputs

- Multiplexed IMC image stacks per imaging region (lung adenocarcinoma TMA BS04081a, healthy tonsil, human endometrium) from Zenodo doi:10.5281/zenodo.6784251
- DNA-intercalator/Histone 3 nuclear channel images for Cellpose nuclei segmentation
- Membrane and cytosolic protein marker channels (metal-isotope conjugated antibody IMC panels) for Mesmer whole-cell expansion

## Outputs

- Whole-cell/cytosol segmentation masks per imaging region
- Verified segmentation-mask visualizations per region
- Segmented single-cell counts per tissue (cell_counts.csv: 19507 lung cancer, 31156 tonsil, 8215 endometrium)

## How to apply

Run Cellpose nuclei segmentation on the DNA-intercalator/Histone 3 nuclear channel of each multiplexed IMC imaging region from the Zenodo deposit (doi:10.5281/zenodo.6784251), then apply the Mesmer pipeline via the scSpaMet wrapper (notebook 11 'Segmentation of single-cell with Mesmer pipeline') to expand nuclei into whole-cell/cytosol masks using membrane and cytosolic protein markers. Visualize the resulting segmentation masks per imaging region (notebook 03) to verify nuclear and whole-cell boundary quality before downstream intensity extraction. The rationale is that nuclear masks alone lose cytosolic protein and metabolite signal, whereas whole-cell masks support single-cell extraction of protein and metabolite expression levels and spatial locations across the lung cancer TMA, tonsil, and endometrium tissues.

## Related tools

- **scSpaMet** (Framework wrapping the segmentation workflow; notebooks 03 (mask visualization) and 11 (Mesmer segmentation) drive nuclei-to-whole-cell mask generation) — github.com/coskunlab/ScSpaMet
- **Mesmer** (Deep-learning segmentation pipeline used to expand Cellpose nuclear masks into whole-cell/cytosol masks from membrane and cytosolic protein markers) — github.com/coskunlab/ScSpaMet
- **Cellpose** (Nuclei segmentation on the DNA-intercalator/Histone 3 IMC channel, providing seeds for Mesmer whole-cell expansion) — github.com/coskunlab/ScSpaMet

## Examples

```
jupyter notebook notebooks/11_Segmentation_of_single_cell_with_Mesmer_pipeline.ipynb
```

## Evaluation signals

- Per-tissue segmented single-cell counts reproduce 19507 (lung cancer), 31156 (tonsil), and 8215 (endometrium) within a reasonable range
- Visual inspection of segmentation masks per imaging region shows nuclear and whole-cell boundaries correctly encompassing cells (notebook 03 output)
- Cytosol masks capture membrane/cytosolic protein marker signal that nuclear-only masks exclude
- Resulting masks support downstream single-cell extraction of protein and metabolite expression levels and spatial locations after cross-modality registration

## Limitations

- TOF-SIMS offers relatively small imaging regions from tissue samples due to the trade-off between imaging speed and spatial resolution, bounding the number of segmentable cells per region
- OCT-embedded frozen tissues preserve metabolites better but have less preserved tissue structures, which can degrade segmentation boundaries
- Nuclear-seeded whole-cell expansion can merge or split touching cells in spatially crowded tissues if membrane markers are weak

## Evidence

- [results] Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
- [abstract] profile cell types and spatial metabolomic maps in three tissues: "profile cell types and spatial metabolomic maps of 19507, 31156, and 8215 single cells in human lung cancer, tonsil, and endometrium tissues, respectively"
- [readme] README notebook 11 covers Mesmer segmentation: "- 11 Segmentation of single-cell with Mesmer pipeline"
- [readme] README notebook 03 covers segmentation mask visualization: "- 03 Single-cell level segmentation and visualization of segmentation masks"
- [readme] article raw data availability in README: "You can find the raw data here: https://doi.org/10.5281/zenodo.6784251."
