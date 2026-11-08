---
name: modification-site-likelihood-scoring
description: Use when you have MS/MS spectra of a structurally related known-unknown
  compound pair and need to score each atom of the known compound for being the modification
  site.
when_to_use_negative:
- The known and unknown spectra are not structurally related analogs (no MS/MS analog-search
  relationship or shared molecular-network edge)
- The pair has more than one modification site (benchmark eligibility requires exactly
  one modification site determined by RDKit GetSubstructMatch edge count) or precursor
  mass difference exceeds 50% of the smaller compound
- The goal is full de novo structure elucidation rather than localizing a modification
  on a known scaffold (use MetFrag, MAGMa, MIDAS, or MS-Finder for general fragmentation
  analysis)
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ModiFinder
  role: Core tool performing MS/MS peak alignment, substructure annotation, and per-atom
    modification-site likelihood scoring; available as CLI and web interface
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: Provides combinatorial fragmentation (breadth-first, depth 2) used to assign
    candidate substructures to each MS/MS peak
- name: SIRIUS
  role: Predicted fragment formulas from SIRIUS (v5.6.3 runnable script) are used
    to filter the possible substructures for each MS/MS peak
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_008
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:661795913fd5c2b08bcaaae732afc3cbdda331c2a435d2b8028b433a3f23dea2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MAGMa
  - ModiFinder
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# modification-site-likelihood-scoring

## Summary

ModiFinder-style per-atom likelihood scoring that localizes structural modification sites by combining MS/MS spectral alignment (shifted vs. unshifted peaks) with MAGMa combinatorial-fragmentation substructure annotations on the known compound's 2D graph.

## When to use

Use when you have MS/MS spectra of a structurally related known-unknown compound pair and need to score each atom of the known compound for being the modification site.

## When NOT to use

- The known and unknown spectra are not structurally related analogs (no MS/MS analog-search relationship or shared molecular-network edge)
- The pair has more than one modification site (benchmark eligibility requires exactly one modification site determined by RDKit GetSubstructMatch edge count) or precursor mass difference exceeds 50% of the smaller compound
- The goal is full de novo structure elucidation rather than localizing a modification on a known scaffold (use MetFrag, MAGMa, MIDAS, or MS-Finder for general fragmentation analysis)

## Inputs

- MS/MS spectra of a known compound and an unknown structural analog (library or experimental, e.g. mzML/mgf)
- 2D chemical structures of the known and unknown compounds (for MAGMa fragmentation and RDKit substructure matching)
- Optional: helper compounds sharing library/adduct/instrument that differ at exactly one site
- Optional: predicted fragment formulas from SIRIUS or BUDDY for substructure filtering

## Outputs

- Per-atom likelihood distribution over the known compound's atoms identifying the putative modification site
- Heatmap visualization of the likelihood landscape on the compound structure (draw_prediction)
- Evaluation scores against the true modification site (e.g. Average-distance Evaluation Score)

## How to apply

Preprocess each spectrum by removing peaks below 1% of the base peak intensity and normalizing intensities to a Euclidean norm of 1. Align MS2-known and MS2-unknown peaks with a greedy bipartite-graph matching (GNPS-style, 40 ppm tolerance) to obtain shifted and unshifted peak matches, since shifted peaks putatively represent substructures containing the modification site. Annotate MS2-known peaks with MAGMa combinatorial fragmentation (breadth-first, maximum depth 2), assigning substructures to peaks whose theoretical m/z falls within 40 ppm. Increase likelihood scores of atoms in assigned substructures for shifted peaks and decrease them for unshifted peaks, optionally weighting by Shannon-entropy clarity; post-process by normalizing, zeroing values below 0.5, raising to the power of 4, and renormalizing to highlight high-scoring atoms. Optionally refine substructure assignments using helper compounds or predicted formulas (SIRIUS/BUDDY) before scoring.

## Related tools

- **ModiFinder** (Core tool performing MS/MS peak alignment, substructure annotation, and per-atom modification-site likelihood scoring; available as CLI and web interface) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (Provides combinatorial fragmentation (breadth-first, depth 2) used to assign candidate substructures to each MS/MS peak)
- **SIRIUS** (Predicted fragment formulas from SIRIUS (v5.6.3 runnable script) are used to filter the possible substructures for each MS/MS peak) — https://github.com/sirius-ms/sirius

## Examples

```
from modifinder import ModiFinder; mf = ModiFinder(known_spectrum, unknown_spectrum, known_structure, unknown_structure, ppm_tolerance=40, fragmentation_depth=2); scores = mf.run()
```

## Evaluation signals

- Benchmark mean Average-distance Evaluation Scores must reproduce MF-N 0.603, MF-R 0.625, and MF-O 0.652 (Fig. 3-A) on the 8033-pair filtered benchmark
- Case-study baselines: Kirromycin N-methylation scores 0.86 before and 0.93 after expert peak-to-fragment-map refinement; Naphthomycin B methylation scores 0.56 before and 0.72 after, with high-scoring sites reduced from 24 to 2
- Score post-processing invariants: values below 0.5 are zeroed and the normalized distribution is raised to the power of 4, so the final distribution sums to 1 with sparse high-scoring atoms
- Unknown 328.211 m/z from Synechococcus elongatus must localize consistently with a shortened lipid tail relative to Glu-C16:1 and a modified amine headgroup relative to Thr-C12:1, matching the manually verified Glu-C12:1

## Limitations

- Scoring quality depends on shifted peaks being explainable by a substructure of the parent compound; pairs lacking such shifted peaks are excluded, leaving only ~62% (8033 pairs) of the benchmark
- MAGMa substructure annotations can be highly ambiguous (e.g. peaks 133.07 and 147.08 m/z of Naphthomycin B matched 27 and 40 substructures respectively), often requiring expert manual peak-to-fragment map curation
- Only a single modification site is assumed; the Oracle variant (MF-O) uses the true modification site and therefore reflects an upper bound rather than realistic performance

## Evidence

- [readme] The documentation is available at: https://wang-bioinformatics-lab.github.io/ModiFinder_base/: [evidence span withheld — anchor status: not_found]
