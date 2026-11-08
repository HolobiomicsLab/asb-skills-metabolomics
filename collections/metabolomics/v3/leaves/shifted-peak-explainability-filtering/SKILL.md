---
name: shifted-peak-explainability-filtering
description: Use when filtering analog compound pairs from GNPS MS/MS libraries to
  retain only pairs whose shifted matched peaks can be annotated by MAGMa substructures,
  before modification-site localization.
when_to_use_negative:
- Pairs that have already been annotated by ModiFinder and need site-localization
  scoring — this skill is a pre-filter, not a scoring method
- Spectral pairs without a single known compound structure, since MAGMa combinatorial
  fragmentation requires the S-known structure to enumerate candidate substructures
- Peaks with intensities below 1% of the base peak, which are removed during preprocessing
  before substructure assignment
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ModiFinder
  role: Provides the spectral alignment, peak preprocessing, and MAGMa substructure-annotation
    logic used to decide whether a shifted peak is explainable
  repo: github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: Performs breadth-first combinatorial fragmentation (maximum depth 2) on the
    known structure to generate candidate substructures for peak annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:b5c32a1b28f32d391a671d417a32821ab1c0638ec19cd00d0a9351113dea5498
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MAGMa
  - ModiFinder
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# shifted-peak-explainability-filtering

## Summary

Filters analog MS/MS compound pairs to keep only those where at least one shifted peak in the spectral alignment is explainable by a substructure of the known compound, per the ModiFinder benchmark.

## When to use

Use when filtering analog compound pairs from GNPS MS/MS libraries to retain only pairs whose shifted matched peaks can be annotated by MAGMa substructures, before modification-site localization.

## When NOT to use

- Pairs that have already been annotated by ModiFinder and need site-localization scoring — this skill is a pre-filter, not a scoring method
- Spectral pairs without a single known compound structure, since MAGMa combinatorial fragmentation requires the S-known structure to enumerate candidate substructures
- Peaks with intensities below 1% of the base peak, which are removed during preprocessing before substructure assignment

## Inputs

- Benchmark analog pairs database from GNPS MS/MS libraries (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB) with pre-verified pair eligibility
- MS2-known and MS2-unknown spectra for each analog pair
- Known compound structures (SMILES) for substructure annotation

## Outputs

- filtered_pairs.csv — the subset of analog pairs passing the shifted-peak explainability criterion
- Retained pair count (8,033 pairs, 62% of 12,909 benchmark pairs)

## How to apply

Start from the benchmark database of analog pairs built from GNPS libraries (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB) that already pass eligibility checks: precursor mass < 2000 Da, precursor mass difference < 50%, same M+H adduct, and exactly one modification site. For each pair, align MS2-known and MS2-unknown spectra using the GNPS bipartite-graph greedy matching at 40 ppm tolerance to classify matched peaks as shifted or unshifted. Preprocess peaks by removing those with intensities below 1% of the base peak and normalizing to a Euclidean norm of 1. Annotate MS2-known peaks with candidate substructures via MAGMa combinatorial fragmentation (breadth-first, maximum depth 2), assigning a substructure when its theoretical m/z falls within 40 ppm of the peak's empirical m/z. Retain a pair only if at least one shifted peak receives at least one substructure assignment; write the retained pairs to filtered_pairs.csv. The expected outcome on the 12,909-pair benchmark is 8,033 retained pairs, i.e. 62% of the total.

## Related tools

- **ModiFinder** (Provides the spectral alignment, peak preprocessing, and MAGMa substructure-annotation logic used to decide whether a shifted peak is explainable) — github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (Performs breadth-first combinatorial fragmentation (maximum depth 2) on the known structure to generate candidate substructures for peak annotation)

## Examples

```
python modidfinder_filter.py --pairs benchmark_pairs.mgf --known-structures structures.smi --out filtered_pairs.csv
```

## Evaluation signals

- Retained pair count on the standard benchmark equals 8,033 pairs (62% of 12,909)
- Every retained pair has at least one shifted peak with a non-empty MAGMa substructure assignment
- All retained pairs previously satisfied eligibility checks: precursor mass < 2000 Da, precursor mass difference < 50%, same M+H adduct, exactly one modification site
- Peaks used for annotation have intensities ≥ 1% of the base peak and are normalized to a Euclidean norm of 1
- Substructure-to-peak assignments fall within a 40 ppm m/z tolerance

## Limitations

- The 40 ppm tolerance can produce spurious substructure assignments, which is why the filter is only the first step before full ModiFinder scoring
- Combinatorial fragmentation is capped at maximum depth 2, so deeper fragmentation pathways are not explored
- Oracle filtering that removes substructures not containing the true modification site is only possible during benchmark evaluation, not on real unknowns
- Pairs failing the eligibility checks (precursor mass < 2000 Da, mass difference < 50%, same M+H adduct, single modification site) are excluded before this filter can be applied

## Evidence
