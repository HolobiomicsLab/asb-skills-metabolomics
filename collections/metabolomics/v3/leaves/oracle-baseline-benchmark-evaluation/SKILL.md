---
name: oracle-baseline-benchmark-evaluation
description: Use when benchmarking modification-site localization variants on the
  ModiFinder GNPS-library benchmark and you need to reproduce or compare MF-N, MF-R,
  and MF-O mean Average-distance scores.
when_to_use_negative:
- The goal is de novo structural elucidation rather than evaluating modification-site
  localization against known true sites — the oracle variant MF-O uses the true modification
  site and cannot be used for real predictions.
- Spectra are not from structurally related known/unknown analog pairs with an identical
  M+H adduct and exactly one modification site (RDKit GetSubstructMatch edge count),
  since benchmark eligibility checks would filter them out.
- Precursor masses exceed 2000 Da or the precursor mass difference is ≥ 50% of the
  smaller compound, violating the benchmark eligibility criteria.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ModiFinder
  role: Runs the MF-N/MF-R/MF-O per-atom likelihood scoring and benchmark evaluation
    variants
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: Combinatorial fragmentation annotation of MS2-known peaks with candidate substructures
    (breadth-first, max depth 2)
- name: MS/MS library search
  role: Provides the GNPS library spectra from which benchmark modified compound pairs
    are assembled
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:79b02aee48fe74b7d8436e051efb168532b414bac33b4d3a3d3ce6d4d0999dbe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MAGMa
  - MS/MS library search
  - ModiFinder
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# oracle-baseline-benchmark-evaluation

## Summary

Reproduce the ModiFinder benchmark evaluation of the three variants MF-N, MF-R, and MF-O by scoring per-atom modification-site likelihoods with the Average-distance evaluation function and matching the reported mean scores of 0.603 / 0.625 / 0.652 (Fig. 3-A).

## When to use

Use when benchmarking modification-site localization variants on the ModiFinder GNPS-library benchmark and you need to reproduce or compare MF-N, MF-R, and MF-O mean Average-distance scores.

## When NOT to use

- The goal is de novo structural elucidation rather than evaluating modification-site localization against known true sites — the oracle variant MF-O uses the true modification site and cannot be used for real predictions.
- Spectra are not from structurally related known/unknown analog pairs with an identical M+H adduct and exactly one modification site (RDKit GetSubstructMatch edge count), since benchmark eligibility checks would filter them out.
- Precursor masses exceed 2000 Da or the precursor mass difference is ≥ 50% of the smaller compound, violating the benchmark eligibility criteria.

## Inputs

- Structurally modified compound pairs from GNPS MS/MS spectral libraries (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB) with known structures and true modification sites
- Preprocessed MS2-known and MS2-unknown peak lists (<1% base-peak intensity filter, Euclidean-norm-1 normalization)
- Shifted and unshifted peak matches from GNPS greedy bipartite-graph alignment (40 ppm tolerance)
- MAGMa substructure-to-peak annotations (breadth-first fragmentation, max depth 2, 40 ppm, charge 1)
- Helper compounds for MF-R refinement, and true modification sites as oracle input for MF-O

## Outputs

- Per-atom normalized likelihood scores over the known compound (thresholded at 0.5 and raised to power 4)
- Per-pair Average-distance evaluation scores for MF-N, MF-R, and MF-O
- Mean Evaluation Scores per variant, expected to match MF-N 0.603, MF-R 0.625, MF-O 0.652 (Fig. 3-A)
- Filtered benchmark subset: 8033 pairs (62% of the total) with at least one shifted peak explainable by a substructure of the parent compound

## How to apply

Build the benchmark pair set from the GNPS MS/MS spectral libraries (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB) applying eligibility checks: precursor mass < 2000 Da, precursor mass difference < 50% of the smaller compound, identical M+H adduct, and exactly one modification site (RDKit GetSubstructMatch edge count). Preprocess spectra by removing peaks below 1% of the base peak intensity and normalizing to a Euclidean norm of 1, then align MS2-known/MS2-unknown peaks with the GNPS greedy bipartite-graph matching at 40 ppm tolerance, and annotate MS2-known peaks with MAGMa combinatorial fragmentation (breadth-first, maximum depth 2). Run the three variants — MF-N on raw annotations, MF-R with helper-compound refinement (helpers share library/adduct/instrument, differ at one site, and compounds within 0.5 precursor mass of the unknown are excluded), and MF-O oracle filtering that removes shifted-peak substructures not containing the true modification site. Compute per-atom likelihood scores (shifted peaks rewarded, unshifted penalized, Shannon-entropy clarity weighting), normalize, zero values below 0.5, raise to the power of 4, and renormalize before scoring each prediction with the Average-distance function against the true modification site.

## Related tools

- **ModiFinder** (Runs the MF-N/MF-R/MF-O per-atom likelihood scoring and benchmark evaluation variants) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (Combinatorial fragmentation annotation of MS2-known peaks with candidate substructures (breadth-first, max depth 2))
- **MS/MS library search** (Provides the GNPS library spectra from which benchmark modified compound pairs are assembled)

## Evaluation signals

- Mean Average-distance Evaluation Scores reproduce MF-N 0.603, MF-R 0.625, and MF-O 0.652 as in Fig. 3-A.
- The filtered benchmark retains 8033 pairs (62% of the total) after keeping only pairs with at least one shifted peak explainable by a parent substructure.
- Per-atom score vectors are non-negative, sum-normalized, contain no values in (0, 0.5), and reflect the power-of-4 sharpening of high-scoring atoms.
- MF-O mean score exceeds MF-R, which exceeds MF-N, consistent with progressively stronger substructure filtering.
- Alignment matches (shifted/unshifted) fall within the 40 ppm m/z tolerance with charge assumed 1.

## Limitations

- The MF-O oracle variant uses the true modification site to filter substructures, so it is an upper-bound baseline, not a deployable method.
- Only benchmark pairs with at least one shifted peak explainable by a substructure of the parent compound are evaluated; 38% of pairs are excluded.
- No accession/deposition ID is given for the ModiFinder benchmark database in the extracted context, complicating exact reproduction.
- Likelihood post-processing (zeroing values below 0.5 and raising to power 4) deliberately de-emphasizes low-scoring atoms, which affects score distributions.

## Evidence
