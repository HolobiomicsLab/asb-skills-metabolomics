---
name: metabolite-intensity-trajectory-projection
description: Use when joint single-cell protein and metabolite intensity tables (e.g.,
  scSpaMet notebook 04 output for tonsil B cells) are available and the goal is to
  order cells along a differentiation trajectory (e.g., GC DZ to LZ) and characterize
  metabolite variation along pseudotime.
when_to_use_negative:
- Input is raw IMC/SIMS images rather than per-cell intensity tables — run the registration,
  segmentation, and intensity extraction notebooks (01–04) first.
- Protein and metabolite modalities are not registered to shared single cells — cross-modality
  registration (Histone 3/Intercalator in IMC, Phosphate 79 m/z in TOF-SIMS) is a
  prerequisite for per-cell metabolite projection.
- The goal is static cell-type metabolite comparison or spatial metabolite competition
  between neighboring cells rather than differentiation dynamics — use the metabolite
  region analysis (notebook 06) instead.
tools:
- name: scSpaMet
  role: 'Framework providing the joint protein-metabolite single-cell pipeline: intensity
    extraction (notebook 04) produces the inputs, and notebook 08 performs diffusion
    pseudotime trajectory analysis and metabolite projection.'
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:0c73ee4f77439de3ef32b21bb6ceddb3fc06c278c826ddc38446b5758726dcc2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scSpaMet
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metabolite-intensity-trajectory-projection

## Summary

Projects co-registered SIMS metabolite intensities onto diffusion pseudotime trajectories built from single-cell IMC protein phenotypes (scSpaMet notebook 08), revealing metabolic patterns along cell differentiation paths such as tonsil germinal-center B-cell maturation.

## When to use

Use when joint single-cell protein and metabolite intensity tables (e.g., scSpaMet notebook 04 output for tonsil B cells) are available and the goal is to order cells along a differentiation trajectory (e.g., GC DZ to LZ) and characterize metabolite variation along pseudotime.

## When NOT to use

- Input is raw IMC/SIMS images rather than per-cell intensity tables — run the registration, segmentation, and intensity extraction notebooks (01–04) first.
- Protein and metabolite modalities are not registered to shared single cells — cross-modality registration (Histone 3/Intercalator in IMC, Phosphate 79 m/z in TOF-SIMS) is a prerequisite for per-cell metabolite projection.
- The goal is static cell-type metabolite comparison or spatial metabolite competition between neighboring cells rather than differentiation dynamics — use the metabolite region analysis (notebook 06) instead.

## Inputs

- Single-cell protein feature matrix from IMC (scSpaMet notebook 04 intensity extraction, tonsil B cells)
- Co-registered SIMS metabolite intensity matrix matched one-to-one to the same segmented cells
- Germinal-center B-cell population labels from protein marker gating (e.g., GC DZ, GC LZ, activated CD38-high B cells)

## Outputs

- Per-cell scalar pseudotime score with cells ordered along the differentiation trajectory
- Metabolite intensity profiles along pseudotime (metabolite trends as a function of maturation state)
- Combined pseudotime-plus-metabolite table, e.g., pseudotime_metabolites_tonsil_gc.csv (one row per GC B cell with pseudotime and metabolite columns)

## How to apply

Load the single-cell protein feature matrix and co-registered SIMS metabolite intensity matrix produced by the scSpaMet single-cell intensity extraction pipeline (notebook 04), and subset to the germinal-center B-cell populations identified by the protein marker panel (e.g., CD38-based activation status). Run trajectory analysis (notebook 08): build a diffusion map on the protein feature matrix and compute diffusion pseudotime to order cells along maturation trajectories (GC DZ B-cells to GC LZ B-cells, and GC DZ B-cells to activated B cells with higher CD38 expression). Assign each cell a scalar pseudotime score, sort cells by pseudotime, and map the SIMS-derived metabolite intensities onto the ordered cells to obtain metabolite profiles along the differentiation path. Write the combined pseudotime-plus-metabolite table (e.g., pseudotime_metabolites_tonsil_gc.csv) and inspect metabolite trends along the pseudotime axis.

## Related tools

- **scSpaMet** (Framework providing the joint protein-metabolite single-cell pipeline: intensity extraction (notebook 04) produces the inputs, and notebook 08 performs diffusion pseudotime trajectory analysis and metabolite projection.) — github.com/coskunlab/ScSpaMet

## Examples

```
jupyter notebook notebooks/08_Trajectory_analysis.ipynb  # run after notebook 04 intensity extraction; output pseudotime_metabolites_tonsil_gc.csv
```

## Evaluation signals

- Output table (e.g., pseudotime_metabolites_tonsil_gc.csv) exists and contains one row per GC B cell with a pseudotime column plus metabolite intensity columns.
- Each cell's SIMS metabolite intensities in the trajectory table match the co-registered intensity matrix from notebook 04 (no row drift between modalities).
- Metabolite intensities vary smoothly (monotonic or non-random trends) along the pseudotime axis, consistent with the reported finding that trajectory inference shows metabolic patterns along differentiation paths.
- Pseudotime ordering recapitulates the expected GC B-cell progression: GC DZ B-cells at early pseudotime and GC LZ B-cells / CD38-high activated B cells at later pseudotime.
- Both differentiation branches (GC DZ→LZ and GC DZ→activated B cells) are recoverable from the diffusion map on the protein feature matrix.

## Limitations

- TOF-SIMS metabolite annotation and coverage is limited for lipids, amino acids, and metabolic pathways, constraining which metabolites can be interpreted along the trajectory.
- Relatively small imaging regions from tissue samples (a trade-off between imaging speed and spatial resolution in TOF-SIMS) limit the number of cells available per trajectory.
- Metabolite preservation and tissue structure depend on sample preparation; OCT-embedded frozen tissues preserve metabolites better than FFPE but with less preserved tissue structure.

## Evidence

- [abstract] Trajectory inference showed metabolic patterns along cell differentiation paths: "Trajectory inference showed metabolic patterns along cell differentiation paths"
- [results] Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
- [readme] 04 Single-cell level intensity extraction and single-cell proteomics clustering: "04 Single-cell level intensity extraction and single-cell proteomics clustering"
- [readme] 08 Trajectory analysis: ""notebooks" folder contains jupyter notebook script used: - 01 Processing of IMC (protein) and SIMS (metabolite) images"
