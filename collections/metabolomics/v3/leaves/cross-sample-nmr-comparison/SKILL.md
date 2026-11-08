---
name: cross-sample-nmr-comparison
description: Use when you have per-sample spin system feature tables (from combined
  TOCSY/HSQC peak picking) for a sample set and need pairwise sample similarity to
  drive network-based dereplication or bioactive metabolite prioritization.
when_to_use_negative:
- Input is raw spectra rather than spin system feature tables — run MADByTE peak picking,
  filtering (solvent/water removal at ±0.02 ppm in 1H, illegitimate HSQC region removal,
  TOCSY <2.5 ppm quadrant exclusion) and spin system construction first
- Compounds with exclusively shielded 1H signals (e.g. sterols, low MW isoprenoids,
  lipids, volatile terpenes) — TOCSY cross-peaks with both F1 and F2 < 2.5 ppm are
  excluded, so these features will be absent from the comparison
- Single-sample analysis — pairwise comparison requires multiple samples to populate
  the matrix
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3314
tools:
- name: MADByTE
  role: Provides the correlation matrix module that computes directional spin system
    feature overlap scores and outputs the asymmetric similarity matrix (correlation_matrix.csv)
  repo: https://github.com/liningtonlab/MADByTE
- name: Gephi
  role: Consumes the resulting network (graphML export of the similarity matrix) for
    Force Atlas 2 visualization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:788cf2e563aaffd640319f84be074ad7c229b5b114cae2b1f626ed4676d9aa97
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cross-sample-nmr-comparison

## Summary

Compares spin system feature tables between samples in an NMR sample set to build an asymmetric pairwise similarity (correlation) matrix, where each off-diagonal entry takes the higher of the two directional scores (A vs B or B vs A) that defines edges in the downstream chemical similarity network.

## When to use

Use when you have per-sample spin system feature tables (from combined TOCSY/HSQC peak picking) for a sample set and need pairwise sample similarity to drive network-based dereplication or bioactive metabolite prioritization.

## When NOT to use

- Input is raw spectra rather than spin system feature tables — run MADByTE peak picking, filtering (solvent/water removal at ±0.02 ppm in 1H, illegitimate HSQC region removal, TOCSY <2.5 ppm quadrant exclusion) and spin system construction first
- Compounds with exclusively shielded 1H signals (e.g. sterols, low MW isoprenoids, lipids, volatile terpenes) — TOCSY cross-peaks with both F1 and F2 < 2.5 ppm are excluded, so these features will be absent from the comparison
- Single-sample analysis — pairwise comparison requires multiple samples to populate the matrix

## Inputs

- Per-sample spin system feature tables from MADByTE spin_system_construction (directed graphs of TOCSY-derived 1H signals annotated with 13C shifts from HSQC)
- List of samples in the set (e.g. the 85-sample Linington lab prefractionated microbial natural products library)

## Outputs

- correlation_matrix.csv — asymmetric square matrix with one row and one column per sample, off-diagonal entries equal to the higher of the two directional pairwise scores

## How to apply

Load the spin system feature tables produced by the MADByTE spin system construction step for every sample. For each ordered pair of samples (A, B), compute a directional similarity score by matching spin system features between the two samples, scoring each pairwise combination of spin system features by (1H, 13C) pair overlap (overlapping pairs divided by total pairs); compute the reciprocal score (B vs A) for the same pair. Populate the asymmetric square matrix using the higher of the two directional scores for each pair, per the MADByTE edge definition. Assemble the full square matrix with one row and one column per sample and write it to correlation_matrix.csv; this matrix feeds downstream network graph generation (exported as graphML for Gephi visualization).

## Related tools

- **MADByTE** (Provides the correlation matrix module that computes directional spin system feature overlap scores and outputs the asymmetric similarity matrix (correlation_matrix.csv)) — https://github.com/liningtonlab/MADByTE
- **Gephi** (Consumes the resulting network (graphML export of the similarity matrix) for Force Atlas 2 visualization)

## Examples

```
conda activate madbyte; python madbyte_gui.py  # run correlation_matrix module on spin system feature tables to emit correlation_matrix.csv
```

## Evaluation signals

- correlation_matrix.csv exists and parses as a square matrix with row and column counts equal to the number of input samples
- Diagonal entries correspond to self-similarity (maximal score); off-diagonal entries reflect the higher of the two directional pairwise scores
- Recomputing the reciprocal score (B vs A) for any pair and taking the max reproduces the stored symmetric-valued entry
- Downstream graphML network in Gephi shows edges consistent with matrix entries; known related compounds (e.g. azithromycin, erythromycin, roxithromycin) cluster via shared spin system features such as the lactone junction (four resonances at 0.76–0.80, 1.37–1.38, 1.77–1.81, 4.75–5.13 ppm)

## Limitations

- MADByTE will not identify every spin system in a given mixture, so similarity scores only cover detected features
- Dereplication depends on user-supplied reference spectra; compound classes with exclusively shielded 1H signals (sterols, lipids, volatile terpenes) are precluded since TOCSY cross-peaks with both dimensions < 2.5 ppm are excluded
- Peak-picking algorithms perform poorly in the crowded shielded region, degrading feature quality that feeds the comparison

## Evidence
