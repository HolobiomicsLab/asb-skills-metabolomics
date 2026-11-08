---
name: asymmetric-similarity-matrix-construction
description: Use when spin system feature tables (from TOCSY+HSQC peak picking) are
  available for a set of NMR samples and a pairwise sample similarity matrix is needed
  as input for chemical similarity network generation.
when_to_use_negative:
- 'Upstream spin system feature tables have not yet been built — run the TOCSY/HSQC
  spin system construction step first (MADByTE pipeline order: data acquisition, peak
  picking, spin system creation, then sample comparison)'
- The sample set contains only compounds with exclusively shielded 1H signals (e.g.
  sterols, low-MW isoprenoids, volatile terpenes) since MADByTE excludes TOCSY cross-peaks
  where F1 and F2 are both < 2.5 ppm
- The goal is pairwise compound-level spectral matching rather than sample-level network
  construction
tools:
- name: MADByTE
  role: Implements the correlation matrix module that scores pairwise spin system
    feature overlap and applies the higher-directional-score edge definition
  repo: https://github.com/liningtonlab/MADByTE
- name: Gephi
  role: 'Downstream consumer: the resulting networks are exported in graphML format
    and visualized in Gephi using the Force Atlas 2 algorithm'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:6143ee87baa2f3a9738487eba0ea60b4aeedb603ca224057474281f1d4c4d42d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# asymmetric-similarity-matrix-construction

## Summary

Constructs the MADByTE sample-pairwise similarity matrix by scoring each ordered pair of samples on (1H, 13C) spin system feature overlap, then resolving the asymmetry with the higher of the two directional scores (A vs B vs B vs A) to define network edges. This is the matrix that feeds chemical similarity network generation for dereplication and metabolite prioritization.

## When to use

Use when spin system feature tables (from TOCSY+HSQC peak picking) are available for a set of NMR samples and a pairwise sample similarity matrix is needed as input for chemical similarity network generation.

## When NOT to use

- Upstream spin system feature tables have not yet been built — run the TOCSY/HSQC spin system construction step first (MADByTE pipeline order: data acquisition, peak picking, spin system creation, then sample comparison)
- The sample set contains only compounds with exclusively shielded 1H signals (e.g. sterols, low-MW isoprenoids, volatile terpenes) since MADByTE excludes TOCSY cross-peaks where F1 and F2 are both < 2.5 ppm
- The goal is pairwise compound-level spectral matching rather than sample-level network construction

## Inputs

- Spin system feature tables per sample (from MADByTE TOCSY/HSQC spin system construction)
- List of samples in the set

## Outputs

- correlation_matrix.csv — asymmetric n x n square matrix, one row and one column per sample, off-diagonal entries equal to the higher of the two directional pairwise feature-overlap scores

## How to apply

Load the spin system feature tables produced for every sample in the set (spin systems are directed graphs built from TOCSY peak tables with 13C shifts annotated from HSQC). For each ordered pair of samples (A, B), compute a directional similarity score as the number of overlapping (1H, 13C) pairs divided by the total number of pairs, then compute the reciprocal score (B vs A). Populate an n x n square matrix (one row and one column per sample) in which each off-diagonal entry takes the higher of the two directional scores, per the MADByTE edge definition. Write the result to correlation_matrix.csv; downstream Gephi visualization of these networks uses graphML export and the Force Atlas 2 algorithm.

## Related tools

- **MADByTE** (Implements the correlation matrix module that scores pairwise spin system feature overlap and applies the higher-directional-score edge definition) — https://github.com/liningtonlab/MADByTE
- **Gephi** (Downstream consumer: the resulting networks are exported in graphML format and visualized in Gephi using the Force Atlas 2 algorithm)

## Examples

```
conda activate madbyte && python madbyte_gui.py
```

## Evaluation signals

- Output file correlation_matrix.csv exists and parses as a square matrix with row and column counts equal to the number of input samples
- Every off-diagonal entry satisfies entry[i][j] == max(score(A vs B), score(B vs A)) — i.e. each cell reflects the higher of the two directional pairwise scores
- Each off-diagonal value lies in (0, 1] since it is an overlap ratio (overlapping pairs / total pairs)
- Matrix is consistent with network edges: the same max-rule value should reproduce the edge weights in the exported graphML network
- Matrix diagonal is populated (self-similarity) and no cells are missing for any sample pair

## Limitations

- MADByTE will not identify every spin system in a given mixture, so the matrix understates similarity for features missed by peak picking
- TOCSY cross-peaks where F1 and F2 are both < 2.5 ppm are excluded, so similarities driven by shielded-region features are absent from the matrix
- Peak-picking algorithms perform poorly in the crowded shielded region, degrading score reliability there
- The matrix is intentionally asymmetric in construction (directional scores differ); the max-rule is a design choice of the MADByTE edge definition, not a symmetric correlation

## Evidence

- [results] The higher of the two similarity scores (A vs B or B vs A) is used to define edges between spin system nodes: "The higher of the two similarity scores (A vs B or B vs A) is used to define edges between spin system nodes"
- [abstract] matches spin system features between samples to create a chemical similarity network: "This platform employs a combination of TOCSY and HSQC spectra to identify spin system features within complex mixtures, and then matches spin system features between samples to create a chemical"
