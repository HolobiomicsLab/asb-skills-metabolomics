---
name: diffusion-pseudotime-trajectory-inference
description: Use when joint protein-metabolite single-cell data (e.g., co-registered
  IMC protein and TOF-SIMS/SIMS metabolite matrices from tonsil germinal centers)
  should be ordered along a maturation trajectory and metabolite trends computed along
  pseudotime.
when_to_use_negative:
- Input cells have not been segmented or intensities not yet extracted at single-cell
  level — run the scSpaMet segmentation/intensity pipeline (notebooks 03–04) first
- Protein and metabolite modalities are not co-registered across IMC and SIMS images
  — the cross-modality registration step must precede trajectory projection
- The biological question is unsupervised metabolite-state discovery rather than differentiation
  ordering — the VAE joint embedding notebook (05) is the appropriate analysis instead
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: scSpaMet
  role: Provides the joint protein-metabolite single-cell framework whose trajectory-analysis
    notebook (08) performs diffusion pseudotime ordering and metabolite projection
    along differentiation paths
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:6cfcc6ccfd84cee3cb0388d7a6c1202123b019f4c15306d89d78191a785c555e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scSpaMet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# diffusion-pseudotime-trajectory-inference

## Summary

Reconstructs B-cell differentiation trajectories (germinal-center dark-zone to light-zone B cells, and GC DZ to activated CD38-high B cells) by applying diffusion pseudotime to single-cell protein phenotypes, then projecting co-registered SIMS metabolite intensities onto the ordered cells to reveal metabolic patterns along differentiation paths.

## When to use

Use when joint protein-metabolite single-cell data (e.g., co-registered IMC protein and TOF-SIMS/SIMS metabolite matrices from tonsil germinal centers) should be ordered along a maturation trajectory and metabolite trends computed along pseudotime.

## When NOT to use

- Input cells have not been segmented or intensities not yet extracted at single-cell level — run the scSpaMet segmentation/intensity pipeline (notebooks 03–04) first
- Protein and metabolite modalities are not co-registered across IMC and SIMS images — the cross-modality registration step must precede trajectory projection
- The biological question is unsupervised metabolite-state discovery rather than differentiation ordering — the VAE joint embedding notebook (05) is the appropriate analysis instead

## Inputs

- Single-cell protein feature matrix from multiplexed IMC imaging of tonsil B cells (scSpaMet notebook 04 output)
- Co-registered SIMS/TOF-SIMS metabolite intensity matrix at single-cell level (scSpaMet single-cell intensity pipeline)
- Germinal-center B-cell population labels from protein marker panel (GC DZ, GC LZ, activated CD38-high B cells)

## Outputs

- Per-cell pseudotime scores ordering GC B cells along the maturation trajectory
- Metabolite profiles plotted along the pseudotime axis for each differentiation path
- Combined pseudotime-plus-metabolite table (e.g., pseudotime_metabolites_tonsil_gc.csv) with one row per GC B cell

## How to apply

Load the single-cell protein feature matrix and the co-registered metabolite intensity matrix produced by the scSpaMet single-cell intensity extraction pipeline (README notebook 04). Subset to germinal-center B-cell populations identified with the IMC protein marker panel. Build a diffusion map on the protein feature matrix using the diffusion pseudotime algorithm implemented in the trajectory analysis notebook (README notebook 08), assign each cell a scalar pseudotime score, and sort cells along the maturation axis. Project the SIMS-derived metabolite intensities onto the pseudotime-ordered cells to obtain metabolite profiles along the GC DZ-to-LZ and GC DZ-to-activated B-cell paths. Write the combined pseudotime-plus-metabolite table (e.g., pseudotime_metabolites_tonsil_gc.csv) and inspect whether metabolite intensities change smoothly rather than randomly along pseudotime, as the paper reports for trajectory-inferred metabolic patterns.

## Related tools

- **scSpaMet** (Provides the joint protein-metabolite single-cell framework whose trajectory-analysis notebook (08) performs diffusion pseudotime ordering and metabolite projection along differentiation paths) — github.com/coskunlab/ScSpaMet

## Evaluation signals

- Output table (e.g., pseudotime_metabolites_tonsil_gc.csv) exists and contains one row per GC B cell with a pseudotime column and metabolite intensity columns
- Metabolite intensities vary smoothly (monotonic/non-random trends) along the pseudotime axis rather than fluctuating randomly
- Reconstructed trajectories reproduce the expected GC DZ-to-LZ and GC DZ-to-activated (CD38-high) B-cell ordering from the protein markers
- Result is consistent with the paper's reported finding that trajectory inference shows metabolic patterns along cell differentiation paths

## Limitations

- Metabolite annotation and pathway coverage from TOF-SIMS (lipids, amino acids) is a stated challenge, limiting biological interpretation of pseudotime-associated metabolites
- Relatively small imaging regions from tissue samples due to the trade-off between imaging speed and spatial resolution in TOF-SIMS restrict trajectory cell counts
- OCT-embedded frozen tissues preserve metabolites better but retain tissue structure less well, which can affect segmentation quality feeding the trajectory

## Evidence

- [abstract] Trajectory inference showed metabolic patterns along cell differentiation paths: "Trajectory inference showed metabolic patterns along cell differentiation paths"
- [results] Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
- [readme] notebooks folder contains jupyter notebook script used: 08 Trajectory analysis: [evidence span withheld — anchor status: not_found]
