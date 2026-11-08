---
name: chemical-similarity-scoring
description: Use when sample-level NMR feature tables (spin systems with 1H/13C shifts)
  from multiple samples must be compared to build a similarity matrix for network
  generation or dereplication.
when_to_use_negative:
- Input is already a network edge list or an already-symmetrized similarity matrix
  — this skill operates on raw spin system feature tables.
- Compounds contain exclusively shielded 1H signals (e.g. sterols, low MW isoprenoids,
  lipids, volatile terpenes) — MADByTE excludes TOCSY cross-peaks where both F1 and
  F2 are < 2.5 ppm, so features for these classes are missing and scores are unreliable.
- Comparing 1D-only spectra or non-TOCSY/HSQC feature representations — the scoring
  depends on paired (1H, 13C) HSQC cross-peaks.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MADByTE
  role: Implements the spin system feature matching and asymmetric correlation matrix
    computation; optionally supports dereplication of the resulting networks against
    a user-supplied internal library.
  repo: https://github.com/liningtonlab/MADByTE
- name: Gephi
  role: Consumes the network derived from the similarity matrix, exported in graphML
    format, for Force Atlas 2 visualization.
- name: NMRglue
  role: Used for visualization of the underlying 1D spectra when inspecting features
    that drive similarity edges.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:2246e8415952930bba1663b8c3cef1e5bc4304a636694776de4a0d368d0fd320
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  - NMRglue
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chemical-similarity-scoring

## Summary

Scores pairwise similarity between samples by matching spin system features (overlapping (1H, 13C) pairs) derived from TOCSY/HSQC data, populating an asymmetric correlation matrix whose edges use the higher of the two directional scores. This drives MADByTE chemical similarity network construction for dereplication and metabolite prioritization.

## When to use

Use when sample-level NMR feature tables (spin systems with 1H/13C shifts) from multiple samples must be compared to build a similarity matrix for network generation or dereplication.

## When NOT to use

- Input is already a network edge list or an already-symmetrized similarity matrix — this skill operates on raw spin system feature tables.
- Compounds contain exclusively shielded 1H signals (e.g. sterols, low MW isoprenoids, lipids, volatile terpenes) — MADByTE excludes TOCSY cross-peaks where both F1 and F2 are < 2.5 ppm, so features for these classes are missing and scores are unreliable.
- Comparing 1D-only spectra or non-TOCSY/HSQC feature representations — the scoring depends on paired (1H, 13C) HSQC cross-peaks.

## Inputs

- Spin system feature tables per sample (1H/13C peak pairs from combined TOCSY and HSQC processing, e.g. correlation_matrix input tables from MADByTE)
- NMR solvent selection used during upstream peak filtering

## Outputs

- correlation_matrix.csv — asymmetric square similarity matrix with one row and one column per sample, off-diagonal entries equal to the higher directional pairwise score

## How to apply

Load the spin system feature tables produced by the spin system construction step for every sample in the set. For each ordered pair of samples (A, B), compute a directional similarity score by matching spin system features, scored as the number of overlapping (1H, 13C) pairs divided by the total pairs; also compute the reciprocal score (B vs A). Populate the asymmetric square matrix using the higher of the two directional scores for each pair, per the MADByTE edge definition, giving one row and one column per sample. Write the result to correlation_matrix.csv; a downstream network step uses this matrix to define spin system nodes and edges (e.g., exported as graphML for Gephi visualization with Force Atlas 2).

## Related tools

- **MADByTE** (Implements the spin system feature matching and asymmetric correlation matrix computation; optionally supports dereplication of the resulting networks against a user-supplied internal library.) — https://github.com/liningtonlab/MADByTE
- **Gephi** (Consumes the network derived from the similarity matrix, exported in graphML format, for Force Atlas 2 visualization.)
- **NMRglue** (Used for visualization of the underlying 1D spectra when inspecting features that drive similarity edges.)

## Examples

```
conda activate madbyte; python madbyte_gui.py
```

## Evaluation signals

- correlation_matrix.csv exists and parses as a square matrix with row and column counts equal to the number of input samples.
- Every off-diagonal entry equals the higher of the two directional pairwise scores (A vs B or B vs A) recomputed from the feature tables.
- Matrix is asymmetric in general — swapping A and B can yield different values, consistent with the MADByTE edge definition.
- Known related compounds (e.g. azithromycin, erythromycin, roxithromycin in the validation set) cluster via shared spin system features such as the lactone junction (0.76–0.80, 1.37–1.38, 1.77–1.81, 4.75–5.13 ppm) and cladinose sugar.
- Diagonal entries are consistent with self-similarity (score of 1) for each sample.

## Limitations

- MADByTE will not identify every spin system in a given mixture, so the similarity matrix reflects only detected features.
- TOCSY cross-peaks where both F1 and F2 chemical shifts are < 2.5 ppm are excluded, precluding analysis of compound classes with exclusively shielded 1H signals (sterols, low MW isoprenoids, lipids, volatile terpenes) unless user-supplied reference spectra are provided.
- Peak-picking algorithms perform poorly in the crowded shielded region, which can degrade feature overlap counts feeding the similarity scores.
- Spectral overlap may yield multiple candidate 13C shifts for a node; the closest HSQC resonance value is used, which can affect feature matching.

## Evidence
