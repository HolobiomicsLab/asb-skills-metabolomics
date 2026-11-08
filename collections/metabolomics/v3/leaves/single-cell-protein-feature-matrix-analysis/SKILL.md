---
name: single-cell-protein-feature-matrix-analysis
description: Use when a co-registered protein (IMC) and metabolite (TOF-SIMS) single-cell
  feature table is available and the goal is cell-population identification plus trajectory/pseudotime
  analysis of differentiation paths.
when_to_use_negative:
- Input images are raw IMC/SIMS frames that have not yet undergone registration and
  single-cell segmentation — run scSpaMet notebooks 01–04 first
- The analysis goal is multi-omics integration across separate modalities without
  single-cell correspondence — the paper notes spatial metabolomic data are unsuitable
  for standard integration methods (MNN, Scanorama, Seurat) because of their untargeted
  nature and lack of shared markers
- Only metabolite data without matched protein profiles are available, since trajectory
  ordering here relies on the protein feature matrix
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_2229
tools:
- name: scSpaMet
  role: Framework providing the single-cell intensity extraction (notebook 04) and
    diffusion pseudotime trajectory analysis (notebook 08) over joint protein-metabolite
    single-cell data
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:62cdac33249489a8172d631fad10384835761ac5d6837ca5ad867acaeb5d9f55
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scSpaMet
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# single-cell-protein-feature-matrix-analysis

## Summary

Analyze single-cell protein feature matrices extracted by the scSpaMet pipeline (notebook 04) to identify cell populations and, via diffusion pseudotime (notebook 08), reconstruct differentiation trajectories with co-registered metabolite profiles along pseudotime.

## When to use

Use when a co-registered protein (IMC) and metabolite (TOF-SIMS) single-cell feature table is available and the goal is cell-population identification plus trajectory/pseudotime analysis of differentiation paths.

## When NOT to use

- Input images are raw IMC/SIMS frames that have not yet undergone registration and single-cell segmentation — run scSpaMet notebooks 01–04 first
- The analysis goal is multi-omics integration across separate modalities without single-cell correspondence — the paper notes spatial metabolomic data are unsuitable for standard integration methods (MNN, Scanorama, Seurat) because of their untargeted nature and lack of shared markers
- Only metabolite data without matched protein profiles are available, since trajectory ordering here relies on the protein feature matrix

## Inputs

- single-cell protein feature matrix from IMC images (scSpaMet notebook 04 intensity extraction)
- co-registered single-cell SIMS metabolite intensity matrix
- protein marker panel annotations for cell-type subsetting (e.g., GC DZ, GC LZ, CD38+ activated B cells)

## Outputs

- per-cell pseudotime scores and trajectory ordering (e.g., GC DZ to LZ, GC DZ to activated B cells)
- combined pseudotime-plus-metabolite table (e.g., pseudotime_metabolites_tonsil_gc.csv) with one row per cell and metabolite columns along pseudotime

## How to apply

Load the single-cell protein feature matrix and the co-registered SIMS metabolite intensity matrix produced by the scSpaMet single-cell intensity extraction pipeline (notebook 04), which follows single-cell segmentation of the registered IMC/SIMS images. Subset to the population of interest using protein markers (e.g., germinal-center B cells in tonsil identified by the protein marker panel, with CD38 marking activated B cells). Build a diffusion map on the protein feature matrix using the diffusion pseudotime algorithm implemented in notebook 08 (DPT/Palantir-style), assign each cell a scalar pseudotime score, and sort cells along the trajectory. Map metabolite intensities onto the ordered cells to profile metabolic variation along the differentiation path (e.g., GC DZ to GC LZ, and GC DZ to activated B cells) and export the combined table (e.g., pseudotime_metabolites_tonsil_gc.csv).

## Related tools

- **scSpaMet** (Framework providing the single-cell intensity extraction (notebook 04) and diffusion pseudotime trajectory analysis (notebook 08) over joint protein-metabolite single-cell data) — github.com/coskunlab/ScSpaMet

## Examples

```
jupyter notebook notebooks/08_Trajectory_analysis.ipynb  # build diffusion map on protein features, export pseudotime_metabolites_tonsil_gc.csv
```

## Evaluation signals

- Output table exists and contains one row per subsetted cell (e.g., GC B cells) with pseudotime and metabolite columns
- Metabolite intensities vary smoothly (monotonic/non-random trends) along the pseudotime axis, consistent with the reported finding that trajectory inference shows metabolic patterns along cell differentiation paths
- Trajectories reconstruct the expected biology: GC DZ to GC LZ and GC DZ to CD38-high activated B cells
- Cell populations are identified by the protein marker panel before trajectory inference (subsetting sanity check)
- Trajectory analysis is run with the scSpaMet notebook 08 script on inputs produced by notebook 04

## Limitations

- TOF-SIMS metabolite annotation and coverage for lipids, amino acids, and metabolic pathways is challenging, which constrains biological interpretation of metabolite trends along pseudotime
- Relatively small imaging regions from tissue samples due to the trade-off between imaging speed and spatial resolution in TOF-SIMS
- Joint analysis requires accurate cross-modality single-cell registration (Histone 3/Intercalator in IMC vs Phosphate 79 m/z in TOF-SIMS); registration errors would propagate into pseudotime-metabolite mapping

## Evidence

- [abstract] Trajectory inference showed metabolic patterns along cell differentiation paths: "Trajectory inference showed metabolic patterns along cell differentiation paths"
- [readme] 08 Trajectory analysis: "- 04 Single-cell level intensity extraction and single-cell proteomics clustering
- 05 VAE joint embedding of protein and metabolite modalities 
- 06 Metabolite analysis"
- [results] applying diffusion pseudotime to GC B-cell protein phenotypes: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
- [results] cross-modality single-cell registration: "a cross-modality single-cell registration pipeline was developed utilizing Histone 3 and Intercalator markers in the IMC dataset, and Phosphate 79 m/z channels in TOF-SIMS"
- [discussion] metabolite annotation challenges from TOF-SIMS: "the challenge in metabolite annotation and coverage for lipids, amino acids, and the metabolic pathways from TOF-SIMS"
