---
name: spin-system-feature-matching
description: Use when you have spin system feature tables (1H/13C shifts with spin
  system IDs) for multiple samples and need a pairwise sample-similarity matrix for
  network/dereplication analysis.
when_to_use_negative:
- Input is still raw or peak-picked TOCSY/HSQC spectra rather than spin system feature
  tables — run spin system construction first.
- Samples contain only exclusively shielded compound classes (sterols, low-MW isoprenoids,
  lipids, volatile terpenes) that the platform precludes from analysis.
- Only a single sample is available — pairwise feature matching requires at least
  two samples.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: MADByTE
  role: Performs the sample-to-sample spin system feature comparison and generates
    the asymmetric correlation matrix (optionally followed by its dereplication module
    and network export)
  repo: https://github.com/liningtonlab/MADByTE
- name: NMRglue
  role: Supports 1D spectra visualization used to QC the TOCSY/HSQC data feeding the
    feature matching
- name: Gephi
  role: Downstream visualization of the network built from the correlation matrix,
    exported as graphML and laid out with Force Atlas 2
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:2fad79169266e64a813917abe4f63e9d08bb5e595d40923f210d27e1c667250a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  - NMRglue
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spin-system-feature-matching

## Summary

Scores pairwise overlap of (1H, 13C) spin system features between samples in a MADByTE analysis set to build the asymmetric correlation matrix that seeds chemical similarity network generation. It operationalizes MADByTE's sample-comparison step, where the higher of two directional scores (A vs B or B vs A) defines each edge.

## When to use

Use when you have spin system feature tables (1H/13C shifts with spin system IDs) for multiple samples and need a pairwise sample-similarity matrix for network/dereplication analysis.

## When NOT to use

- Input is still raw or peak-picked TOCSY/HSQC spectra rather than spin system feature tables — run spin system construction first.
- Samples contain only exclusively shielded compound classes (sterols, low-MW isoprenoids, lipids, volatile terpenes) that the platform precludes from analysis.
- Only a single sample is available — pairwise feature matching requires at least two samples.

## Inputs

- spin system feature tables per sample (spin_system_features.csv: 1H and 13C chemical shifts with spin system IDs) from the TOCSY/HSQC graph construction step
- sample set metadata defining which samples to compare

## Outputs

- correlation_matrix.csv — asymmetric square similarity matrix, one row and one column per sample, off-diagonal entries = higher of the two directional pairwise scores

## How to apply

Load the spin system feature tables produced by the spin system construction step for every sample in the set. For each ordered pair of samples (A, B), compute a directional similarity score by matching spin system features, counting overlapping (1H, 13C) pairs divided by the total number of pairs; compute the reciprocal score (B vs A) for the same pair. Populate an asymmetric square matrix in which each off-diagonal entry takes the higher of the two directional scores, per MADByTE's edge definition, with one row and one column per sample. Write the result to correlation_matrix.csv for downstream network graph generation. Run this inside the MADByTE conda environment after peak picking and spin system construction (which required TOCSY shielded-quadrant exclusion at <2.5 ppm and HSQC filtering of solvent/water bands at ±0.02 ppm and the three illegitimate HSQC regions).

## Related tools

- **MADByTE** (Performs the sample-to-sample spin system feature comparison and generates the asymmetric correlation matrix (optionally followed by its dereplication module and network export)) — https://github.com/liningtonlab/MADByTE
- **NMRglue** (Supports 1D spectra visualization used to QC the TOCSY/HSQC data feeding the feature matching)
- **Gephi** (Downstream visualization of the network built from the correlation matrix, exported as graphML and laid out with Force Atlas 2)

## Examples

```
conda activate madbyte && python madbyte_gui.py
```

## Evaluation signals

- correlation_matrix.csv exists and parses as a square matrix whose row and column counts equal the number of input samples
- each off-diagonal entry equals the higher of the two directional pairwise scores (A vs B or B vs A) recomputed from the feature tables
- directional scores are consistent with overlapping (1H, 13C) pairs divided by total pairs for each ordered sample pair
- known related compounds (e.g. azithromycin, erythromycin, roxithromycin) cluster via shared spin system features such as the lactone junction and cladinose sugar

## Limitations

- MADByTE will not identify every spin system in a given mixture, so matching operates on an incomplete feature set
- TOCSY cross-peaks where both F1 and F2 are < 2.5 ppm are excluded, so compounds with exclusively shielded 1H signals (sterols, low MW isoprenoids, lipids, volatile terpenes) cannot be matched unless users supply reference spectra
- Peak picking performs poorly in the crowded shielded region, and spectral overlap can yield multiple candidate 13C chemical shifts (closest HSQC resonance is used), affecting feature fidelity

## Evidence
