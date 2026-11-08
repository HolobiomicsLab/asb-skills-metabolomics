---
name: tonsil-b-cell-population-subsetting
description: Use when a scSpaMet single-cell feature table (protein + co-registered
  SIMS metabolite intensities) from tonsil tissue must be narrowed to germinal-center
  B cells before pseudotime or metabolite-trajectory analysis.
when_to_use_negative:
- The tissue is lung adenocarcinoma or endometrium rather than tonsil — GC B-cell
  markers are not the relevant gating scheme
- The input is raw IMC/SIMS images that have not yet passed single-cell segmentation
  and intensity extraction (notebooks 03/04)
- The downstream goal is metabolite-competition or spatial-interaction analysis across
  all cell types, not lineage-specific trajectory analysis
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scSpaMet
  role: Provides the single-cell segmentation, intensity-extraction, and proteomics-clustering
    notebooks whose outputs are subsetted to tonsil GC B-cell populations
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:4f30ce132d230849e75c1d9675804d87340bb4705d40279dc89f2be59c75e504
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scSpaMet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# tonsil-b-cell-population-subsetting

## Summary

Isolate germinal-center B-cell populations from scSpaMet single-cell protein-metabolite intensity matrices of healthy human tonsil tissue using the IMC protein marker panel, so that downstream trajectory (DPT/Palantir pseudotime) and metabolite-projection analyses operate only on the relevant cell lineage.

## When to use

Use when a scSpaMet single-cell feature table (protein + co-registered SIMS metabolite intensities) from tonsil tissue must be narrowed to germinal-center B cells before pseudotime or metabolite-trajectory analysis.

## When NOT to use

- The tissue is lung adenocarcinoma or endometrium rather than tonsil — GC B-cell markers are not the relevant gating scheme
- The input is raw IMC/SIMS images that have not yet passed single-cell segmentation and intensity extraction (notebooks 03/04)
- The downstream goal is metabolite-competition or spatial-interaction analysis across all cell types, not lineage-specific trajectory analysis

## Inputs

- Single-cell protein intensity matrix from scSpaMet notebook 04 (IMC-derived, tonsil)
- Co-registered SIMS/TOF-SIMS metabolite intensity matrix for the same cells
- Segmentation masks / per-cell spatial locations from scSpaMet notebook 03
- Protein marker panel annotations (e.g. CD38, GC DZ/LZ markers)

## Outputs

- Filtered protein feature matrix containing only germinal-center B cells
- Row-aligned filtered metabolite intensity matrix for the same GC B cells
- Cell identity/cluster labels marking GC B-cell subpopulations (DZ, LZ, activated CD38-high B cells)

## How to apply

Start from the single-cell protein and metabolite intensity matrices produced by the scSpaMet intensity-extraction notebook (notebook 04), where single-cell segmentation has already yielded per-cell protein and metabolite expression levels and spatial locations. Identify germinal-center B-cell populations using the targeted multiplexed IMC protein marker panel (e.g. CD38 for activated B cells, DZ/LZ markers distinguishing GC dark-zone from light-zone B cells), gating or clustering on the protein channels rather than the SIMS channels. Subset the rows of both the protein feature matrix and the co-registered metabolite matrix to these GC B cells so the two modalities stay row-aligned per cell. The rationale is that trajectory inference (diffusion pseudotime in notebook 08) on mixed populations would confound differentiation order with cell-type composition, so subsetting to GC B cells is a prerequisite for reconstructing the GC DZ-to-LZ and GC DZ-to-activated B-cell paths.

## Related tools

- **scSpaMet** (Provides the single-cell segmentation, intensity-extraction, and proteomics-clustering notebooks whose outputs are subsetted to tonsil GC B-cell populations) — github.com/coskunlab/ScSpaMet

## Evaluation signals

- The subset table contains one row per GC B cell with both pseudotime and metabolite columns populated
- Cell counts of the subset match the expected tonsil GC B-cell fraction of the 31156 profiled tonsil single cells
- Protein-marker gating is verifiable: activated B cells show higher CD38 expression, consistent with the paper's reported GC DZ-to-activated B-cell path
- Downstream diffusion pseudotime orders cells sensibly, and metabolite intensities vary smoothly (monotonic/non-random trends) along the pseudotime axis
- Protein and metabolite matrices remain row-aligned after filtering (same cell indices in both)

## Limitations

- Cell-type identification depends entirely on the 25-protein IMC marker panel; metabolite channels alone cannot label GC B-cell subpopulations
- TOF-SIMS metabolite annotation and coverage for lipids, amino acids, and metabolic pathways is a stated challenge, which limits interpretation of the subsetted metabolite matrix
- The cross-modality registration (IMC to SIMS via Histone 3/Intercalator and Phosphate 79 m/z channels) introduces alignment error that propagates into co-registered single-cell metabolite values

## Evidence

- [results] tonsil GC B-cell subsetting: "The scSpaMet was applied to healthy human tonsil tissues (Supplementary Table 5)"
- [readme] intensity extraction and clustering: "04 Single-cell level intensity extraction and single-cell proteomics clustering"
- [abstract] trajectory basis: "Trajectory inference showed metabolic patterns along cell differentiation paths"
