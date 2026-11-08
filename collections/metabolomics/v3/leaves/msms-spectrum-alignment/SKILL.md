---
name: msms-spectrum-alignment
description: Use when you have MS2 spectra of a known library hit and an unknown analog
  whose precursor masses differ by a partial modification, and you need to determine
  which atoms/substructures carry the modification.
when_to_use_negative:
- The unknown has no structurally related known analog with a matching MS/MS spectrum
  — plain library search is the appropriate tool instead
- The goal is de novo fragmentation-tree analysis without a reference structure (e.g.
  standalone SIRIUS/MetFrag use), rather than localization of a modification between
  two related structures
- Precursor mass difference between the known and unknown exceeds 50% or the precursor
  exceeds 2000 Da — benchmark eligibility checks (i–iv) exclude such pairs
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: ModiFinder
  role: Core tool performing MS/MS spectral alignment between known and unknown analogs
    and scoring per-atom modification-site likelihoods
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: Combinatorial fragmentation approach (breadth-first, depth 2) used to assign
    possible substructures to each aligned MS/MS peak
- name: SIRIUS
  role: Predicted formulas from SIRIUS (v5.6.3 runnable script) are used to filter
    the possible substructures for each MS/MS peak
  repo: https://github.com/boecker-lab/sirius
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:4f2a023d0117ab262d8c91807a4ec8d85c8396631d589af291933b8aa59f2b65
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MAGMa
  - ModiFinder
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# msms-spectrum-alignment

## Summary

Align MS/MS peaks between a structurally related known molecule and an unknown analog to identify shifted and unshifted fragment peaks that localize the structural modification site, as implemented in ModiFinder.

## When to use

Use when you have MS2 spectra of a known library hit and an unknown analog whose precursor masses differ by a partial modification, and you need to determine which atoms/substructures carry the modification.

## When NOT to use

- The unknown has no structurally related known analog with a matching MS/MS spectrum — plain library search is the appropriate tool instead
- The goal is de novo fragmentation-tree analysis without a reference structure (e.g. standalone SIRIUS/MetFrag use), rather than localization of a modification between two related structures
- Precursor mass difference between the known and unknown exceeds 50% or the precursor exceeds 2000 Da — benchmark eligibility checks (i–iv) exclude such pairs

## Inputs

- MS2-known spectrum (MS/MS of the known library structure)
- MS2-unknown spectrum (MS/MS of the unknown analog)
- Precursor m/z values for known and unknown compounds
- Known molecule structure (S-known) for combinatorial fragmentation

## Outputs

- MS/MS peak alignment (bipartite matching of shifted and unshifted peaks)
- Per-atom likelihood scores for the modification site on the known molecule
- Post-processed likelihood heatmap (values <0.5 set to 0, values raised to the power of 4)

## How to apply

Preprocess both MS/MS spectra by removing peaks below 1% of the base peak and normalizing intensities to a Euclidean norm of 1. Build a bipartite graph whose nodes are the peaks of MS2-known and MS2-unknown, and find a greedy matching of peaks within the m/z tolerance (GNPS cosine alignment; 40 ppm tolerance is used for substructure-to-peak matching). Peaks that shift between the two spectra represent substructures containing the modification site; peaks at matching m/z represent substructures that do not. Assign each peak a set of possible substructures via MAGMa-style combinatorial fragmentation (breadth-first, maximum depth 2) on the known structure, then increase the likelihood scores of atoms in substructures assigned to shifted peaks and decrease scores for unshifted peaks. Post-process the per-atom likelihood map by setting values below 0.5 to 0 and raising values to the power of 4 to emphasize high-scoring atoms before interpreting the modification site.

## Related tools

- **ModiFinder** (Core tool performing MS/MS spectral alignment between known and unknown analogs and scoring per-atom modification-site likelihoods) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (Combinatorial fragmentation approach (breadth-first, depth 2) used to assign possible substructures to each aligned MS/MS peak)
- **SIRIUS** (Predicted formulas from SIRIUS (v5.6.3 runnable script) are used to filter the possible substructures for each MS/MS peak) — https://github.com/boecker-lab/sirius

## Evaluation signals

- Per-atom likelihood scores concentrate on the true modification site (localized region consistent with the verified structure, e.g. Glu-C12:1's shortened lipid tail vs Glu-C16:1 and modified amine headgroup vs Thr-C12:1)
- Alignment includes both shifted peaks (covering the modification site) and unshifted peaks; unshifted peaks should drive likelihood decreases at non-modified atoms
- All peaks below 1% of the base peak are absent from the aligned spectra and retained peak intensities sum to a Euclidean norm of 1
- Substructure-to-peak assignments fall within the 40 ppm m/z tolerance
- Post-processed score vector has no values strictly between 0 and 0.5, and high-scoring atoms are amplified by the power-of-4 transform

## Limitations

- Performance and limitations were specifically assessed via 'proximity cover' and 'ambiguity cover' criteria; localization can be ambiguous when multiple substructures fit a shifted peak
- Fragmentation depth is capped at 2 in the combinatorial fragmentation, so deeper fragmentation paths are not explored
- The method requires the known structure (S-known); it cannot localize modifications without a structurally related reference and its MS/MS spectrum
- ModiFinder_base is under active development, per the project README

## Evidence

- [readme] ModiFinder is a tool for site localization of structural modifications using MS/MS data.: [evidence span withheld — anchor status: not_found]
