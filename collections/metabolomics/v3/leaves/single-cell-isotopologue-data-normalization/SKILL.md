---
name: single-cell-isotopologue-data-normalization
description: Use when processing SpaceM single-cell MALDI imaging output (spatiomolecular
  adata per well) into quality-controlled, per-cell fatty acid isotopologue distributions
  ready for binomial labelling fits.
when_to_use_negative:
- Processing mouse brain tissue imaging data, where the same corrections are applied
  per pixel of the MALDI image instead of per segmented cell
- Data that has not been through SpaceM registration/segmentation (single-cell readouts
  depend on ablation marks overlaid with segmentation cell masks)
- Rerunning from raw imzML files when SpaceM outputs are already shared — the README
  notes you don't need to download raw imzML and rerun the ion image extraction step
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: 13C-SpaceM
  role: Repository containing the sc_analysis.py script that performs correction,
    aggregation and normalization of single-cell isotopologue data
  repo: https://github.com/Buglakova/13C-SpaceM
- name: Scanpy
  role: Stores and reads the pickled anndata objects holding the single-cell isotopologue
    data
  repo: https://github.com/Buglakova/13C-SpaceM
- name: pyimzML
  role: Used upstream to extract ion images for all isotopologues of the fatty acids
    from each imzML file in the format expected by SpaceM
  repo: https://github.com/alexandrovteam/pyimzML
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:6109560f0ba726806649924d40e01c6d96aea849a8ed242c80ab3394846dac96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Buglakova/13C-SpaceM
  tools:
  - 13C-SpaceM
  - Scanpy
  - pyimzML
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# single-cell-isotopologue-data-normalization

## Summary

Performs single-cell isotopologue data normalization within the 13C-SpaceM pipeline: natural isotope abundance correction, median aggregation of ablation-mark signal per cell, and normalization of fatty acid isotopologue distributions stored in anndata objects.

## When to use

Use when processing SpaceM single-cell MALDI imaging output (spatiomolecular adata per well) into quality-controlled, per-cell fatty acid isotopologue distributions ready for binomial labelling fits.

## When NOT to use

- Processing mouse brain tissue imaging data, where the same corrections are applied per pixel of the MALDI image instead of per segmented cell
- Data that has not been through SpaceM registration/segmentation (single-cell readouts depend on ablation marks overlaid with segmentation cell masks)
- Rerunning from raw imzML files when SpaceM outputs are already shared — the README notes you don't need to download raw imzML and rerun the ion image extraction step

## Inputs

- SpaceM output directories per well (config.json, ablation_mark_analysis, cell_segmentation_external, ion_image_registration, overlap_analysis2, single_cell_analysis, transformation)
- List of fatty acid ions with all isotopologues extracted from imzML files via extract_isotopologue_ion_images.py (e.g. analysis/ion_list_fa.csv)
- Slide/well directory structure for the experiment (e.g. 20220322_AB_DKFZHypoxia/slide2, 20220411_AB_DKFZACLYac/slide1-3)

## Outputs

- Per-slide anndata (.h5ad) object with corrected, median-aggregated and normalized per-cell fatty acid isotopologue distributions
- Quality-controlled, aggregated anndata objects containing per-cell information for manuscript plots (analysis/hypoxia and analysis/ACLYkd notebooks)

## How to apply

Run `python sciso/sc_analysis.py $data_dir/<experiment>/slide<N>/spacem_data $data_dir/<experiment>/slide<N>/anndata` for each slide, which first applies natural isotope abundance correction for each fatty acid in each ablation mark, then averages signal in each cell by taking a median for each annotated m/z value, and finally normalizes the isotopologue distribution for each fatty acid for each cell. Results are saved as a per-slide anndata object joined across all wells on that slide to keep file sizes manageable. After per-slide analysis, aggregate all slides and perform quality control, filtering out cells by size, number and total intensity of detected ions, and classify cells using normalized GFP intensity. The anndata objects use Scanpy pickled format, so version mismatches among pandas, anndata, pickle and scanpy may need troubleshooting with environment_dump.yml.

## Related tools

- **13C-SpaceM** (Repository containing the sc_analysis.py script that performs correction, aggregation and normalization of single-cell isotopologue data) — https://github.com/Buglakova/13C-SpaceM
- **Scanpy** (Stores and reads the pickled anndata objects holding the single-cell isotopologue data) — https://github.com/Buglakova/13C-SpaceM
- **pyimzML** (Used upstream to extract ion images for all isotopologues of the fatty acids from each imzML file in the format expected by SpaceM) — https://github.com/alexandrovteam/pyimzML

## Examples

```
python sciso/sc_analysis.py $data_dir/20220411_AB_DKFZACLYac/slide1/spacem_data $data_dir/20220411_AB_DKFZACLYac/slide1/anndata
```

## Evaluation signals

- Each slide produces a single anndata object joined across all wells, with natural-isotope-corrected, median-aggregated, normalized isotopologue distributions per fatty acid per cell
- Cells failing QC (by size, number and total intensity of detected ions) are filtered out before downstream plotting
- Downstream binomial fits on the normalized palmitate distributions reproduce reported acetyl-CoA labelling peaks: control p≈0.6, ACLYkd oligo 1 p≈0.35, ACLYkd oligo 2 p≈0.4
- Isotopologue distributions are normalized (sum to 1) per fatty acid per cell, as required as input by fit_binomial in the 13C_binomial repository

## Limitations

- Scanpy pickled anndata objects can break with certain combinations of pandas, anndata, pickle and scanpy versions; environment_dump.yml is provided for troubleshooting
- Long conda environment solve times are a known issue; the mamba solver is recommended
- Analysis is done per slide to keep file sizes manageable, so cross-slide aggregation is a separate step

## Evidence

- [methods] Natural isotope abundance correction, median aggregation, isotopologue normalization: "This script performs the following steps: Natural isotope abundance correction for each fatty acid in each ablation mark; Averaging of signal in each cell by taking a median for each annotated m/z"
- [readme] Per-slide anndata output: "The results are saved in an `anndata` object joined for all wells on the slide. This analysis is done per slide to keep the file sizes manageable."
- [readme] QC filtering criteria: "After single-cell analysis data from all slides is aggregated together and quality control is performed, filtering out cells by size and number and total intensity of detected ions."
- [readme] Scanpy version fragility: "This project uses Scanpy's pickled `anndata` objects to store the single-cell data. Sometimes different combinations of versions of `pandas`, `anndata`, `pickle` and `scanpy` can cause errors"
