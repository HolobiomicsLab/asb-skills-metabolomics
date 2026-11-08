---
name: molecular-formula-enumeration
description: Use when building a custom METASPACE molecular database for annotating
  matrix-cluster signals in MALDI-imaging data, i.e. when you need to systematically
  enumerate all plausible cluster ion formulas for a matrix such as DHB.
when_to_use_negative:
- The goal is annotating biological metabolites of the sample rather than matrix cluster
  signals — use the standard METASPACE databases (e.g. HMDB) instead.
- The matrix is not DHB (C7H6O4) and no analogous cluster-generating model (Keller
  and Li-style) exists for that matrix; the enumeration parameters would not be valid.
- The task is classifying ion images as on- vs off-sample from already-annotated data
  — use the off-sample recognition classifier/biclustering skill, not formula enumeration.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: METASPACE
  role: Cloud platform into which the enumerated DHB matrix cluster formulas were
    loaded as a custom molecular database to annotate MALDI-imaging datasets and surface
    off-sample matrix cluster images.
  repo: https://github.com/metaspace2020/metaspace/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:71746b70c3e912cf542a788c49f42e2b2b90ca3d61cb3dfef2829c73cab7c8df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# molecular-formula-enumeration

## Summary

Enumerates candidate MALDI matrix cluster ion formulas combinatorially by combining intact matrix molecules, water-loss species, and H/K/Na adduct modifications (here M = C7H6O4, 2,5-dihydroxybenzoic acid), yielding a closed molecular database used to annotate matrix clusters in METASPACE.

## When to use

Use when building a custom METASPACE molecular database for annotating matrix-cluster signals in MALDI-imaging data, i.e. when you need to systematically enumerate all plausible cluster ion formulas for a matrix such as DHB.

## When NOT to use

- The goal is annotating biological metabolites of the sample rather than matrix cluster signals — use the standard METASPACE databases (e.g. HMDB) instead.
- The matrix is not DHB (C7H6O4) and no analogous cluster-generating model (Keller and Li-style) exists for that matrix; the enumeration parameters would not be valid.
- The task is classifying ion images as on- vs off-sample from already-annotated data — use the off-sample recognition classifier/biclustering skill, not formula enumeration.

## Inputs

- Cluster-generating model specification (Keller and Li, 2000 reference PDF, ./DHB matrix clusters/Keller and Li, 2000.pdf)
- Base matrix molecular formula, M = C7H6O4 (2,5-dihydroxybenzoic acid, DHB)
- Bounds/valid ranges for the enumeration parameters (n, p, x, y, z) as defined in the model

## Outputs

- CSV file 'DHB mc mol formulas, predicted.csv' containing 353 predicted molecular formulas
- Formula set usable as a custom METASPACE molecular database for matrix-cluster annotation

## How to apply

Load the cluster-generating model specification from the Keller and Li (2000) reference and set the base molecule M = C7H6O4 (2,5-dihydroxybenzoic acid, DHB). Enumerate the combinatorial space of candidate cluster ion formulas of the form n*M + p*(M-H2O) - x*H + y*K + z*Na, combining intact DHB molecules and dehydrated DHB units with variable proton losses and potassium/sodium additions. For each valid (n, p, x, y, z) combination, sum element counts and normalize the resulting chemical formula notation. Export the complete enumerated set — 353 predicted molecular formulas for DHB — to 'DHB mc mol formulas, predicted.csv'. The rationale is that off-sample MALDI images are often polluted by matrix clusters, so annotating them with a dedicated formula database enables their recognition and filtering in METASPACE.

## Related tools

- **METASPACE** (Cloud platform into which the enumerated DHB matrix cluster formulas were loaded as a custom molecular database to annotate MALDI-imaging datasets and surface off-sample matrix cluster images.) — https://github.com/metaspace2020/metaspace/

## Evaluation signals

- Output CSV 'DHB mc mol formulas, predicted.csv' exists, parses correctly, and contains exactly 353 molecular formula rows.
- Every generated formula is decomposable into the model form n*M + p*(M-H2O) - x*H + y*K + z*Na with M = C7H6O4.
- Element counts of each formula equal the sum of constituent species counts before normalization.
- Enumerated formulas are consistent with the space defined in the Keller and Li (2000) model specification.
- No duplicate formulas remain after normalizing chemical formula notation.

## Limitations

- The enumerated set is specific to DHB matrix clusters; off-sample formation for other MALDI matrices is not covered (the paper notes matrix formation 'in general as well as for specific MALDI matrices' was not solved).
- The model enumerates candidate formulas but does not by itself determine which clusters occur or which images are off-sample — that requires off-sample recognition (deep learning F1 0.97, biclustering F1 0.96) and manual expert review.

## Evidence

- [readme] DHB cluster generation per Keller and Li 2000: [evidence span withheld — anchor status: not_found]
- [intro] Derivation of matrix cluster properties from DHB off-sample images: "we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB) [18] and derived properties of matrix clusters"
- [abstract] Characterization of DHB matrix cluster properties: "we investigated off-sample images corresponding to the most common MALDI matrix (2,5-dihydroxybenzoic acid, DHB) and characterized properties of matrix clusters"
