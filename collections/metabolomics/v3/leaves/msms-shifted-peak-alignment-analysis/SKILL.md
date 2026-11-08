---
name: msms-shifted-peak-alignment-analysis
description: Use when two structurally related MS/MS spectra (MS2-known and MS2-unknown)
  are available with a 2D structure of the known compound, and the goal is to localize
  where the structural modification occurs on the known molecule.
when_to_use_negative:
- The unknown compound has no structurally related known compound with matching MS/MS
  spectra — analog library search or molecular networking identification alone is
  the appropriate choice
- No 2D structure is available for the known compound, since combinatorial fragmentation
  requires the initial structure S-known
- Precursor mass difference exceeds the benchmark eligibility criteria (e.g. difference
  in precursor masses >= 50% or precursor mass above 2000 Da)
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: ModiFinder
  role: Performs MS/MS spectral alignment, combinatorial-fragmentation substructure
    assignment, and atom-level modification-site scoring; used via MF-R with MAGMa
    fragmentation at depth 2 and 40 ppm tolerance
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: Combinatorial fragmentation engine whose breadth-first fragmentation (depth
    2) supplies candidate substructures for each MS/MS peak
- name: MS/MS library search
  role: Source of reference spectra for known compounds and analogs (e.g. TUEBINGEN,
    GNPS accessions) retrieved with ModiFinder's Compound/Spectrum loaders
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:a8f4dc943e74bbe3ee60506027f370aebf7755c03d4b6dba6443090ff3dba4dd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MAGMa
  - MS/MS library search
  - ModiFinder
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# msms-shifted-peak-alignment-analysis

## Summary

A ModiFinder-based analysis skill that aligns MS/MS peaks between a known compound and a structural analog to score every atom of the known molecule for its likelihood of being the modification site, optionally refined by expert peak-to-fragment mapping edits.

## When to use

Use when two structurally related MS/MS spectra (MS2-known and MS2-unknown) are available with a 2D structure of the known compound, and the goal is to localize where the structural modification occurs on the known molecule.

## When NOT to use

- The unknown compound has no structurally related known compound with matching MS/MS spectra — analog library search or molecular networking identification alone is the appropriate choice
- No 2D structure is available for the known compound, since combinatorial fragmentation requires the initial structure S-known
- Precursor mass difference exceeds the benchmark eligibility criteria (e.g. difference in precursor masses >= 50% or precursor mass above 2000 Da)

## Inputs

- MS/MS spectra of the known compound and its structural analog (MS2-known and MS2-unknown), e.g. from TUEBINGEN or GNPS library accessions via ModiFinder Compound/Spectrum loaders
- 2D molecular structure of the known compound
- Peak-to-fragment maps (for expert refinement of substructure assignments)
- True modification site annotation, when available for evaluation

## Outputs

- Atom-level likelihood distribution over the known molecule's modification site
- Average-distance Evaluation Scores before and after refinement (e.g. 0.86->0.93 for Kirromycin, 0.56->0.72 for Naphthomycin B)
- High-scoring modification site counts (e.g. reduced from 24 to 2 for Naphthomycin B)
- refinement_scores.csv summarizing pre/post-refinement scores and ambiguity-reduction counts

## How to apply

Preprocess each MS/MS spectrum by removing peaks below 1% of the base peak and normalizing intensities to a Euclidean norm of 1. Build a bipartite graph between the MS2-known and MS2-unknown peaks and match via greedy alignment, then run combinatorial fragmentation (MAGMa-style breadth-first at depth 2) on S-known to assign each peak possible substructures within a 40 ppm m/z tolerance. Run ModiFinder MF-R to produce likelihood distributions over atoms: shifted peaks increase scores of atoms in assigned substructures, unshifted peaks decrease them; final post-processing sets values below 0.5 to 0 and raises scores to the power of 4. Evaluate localization using the Average-distance Evaluation Score (e.g. baselines 0.86 for Kirromycin-Goldinodox and 0.56 for Naphthomycin A-B). When domain knowledge indicates ambiguous fragment assignments (e.g. 133.07 and 147.08 m/z peaks in Naphthomycin B spanning 27 and 40 substructures), restrict peak-to-fragment maps to plausible substructures and recompute refined scores (0.93 and 0.72 expected), counting high-scoring modification sites to confirm ambiguity reduction (24 sites down to 2).

## Related tools

- **ModiFinder** (Performs MS/MS spectral alignment, combinatorial-fragmentation substructure assignment, and atom-level modification-site scoring; used via MF-R with MAGMa fragmentation at depth 2 and 40 ppm tolerance) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (Combinatorial fragmentation engine whose breadth-first fragmentation (depth 2) supplies candidate substructures for each MS/MS peak)
- **MS/MS library search** (Source of reference spectra for known compounds and analogs (e.g. TUEBINGEN, GNPS accessions) retrieved with ModiFinder's Compound/Spectrum loaders)

## Evaluation signals

- Baseline Average-distance Evaluation Scores reproduce published values (0.86 for Kirromycin-Goldinodox, 0.56 for Naphthomycin A-B) using the BasicEvaluationEngine with true modification sites (N-methylation; methylation)
- After expert peak-to-fragment refinement, scores improve to 0.93 and 0.72, and Naphthomycin B high-scoring modification sites drop from 24 to 2
- Peak preprocessing invariant: all peaks below 1% of the base peak removed and normalized intensities sum to a Euclidean norm of 1
- Substructure-to-peak assignments satisfy the 40 ppm m/z tolerance and fragmentation depth of 2
- Post-processed likelihood scores obey the published post-processing: values below 0.5 set to 0, all scores raised to the power of 4

## Limitations

- Fragment-to-substructure assignments can be highly ambiguous (e.g. 27 and 40 candidate substructures for single peaks), so expert editing of peak-to-fragment maps is needed to reduce ambiguity
- Benchmark pairs are restricted by eligibility checks: precursor mass < 2000 Da and precursor-mass difference < 50%
- Helpers-based refinement eliminates compounds whose precursor mass falls within a 0.5 range of the unknown's precursor mass, constraining applicable comparisons

## Evidence

- [readme] skill purpose: [evidence span withheld — anchor status: not_found]
