---
name: cidmd-spectrum-aggregation-statistics
description: Use when you have a cohort of predicted (in silico) MS/MS spectra and
  matched experimental library spectra and need cohort-level accuracy statistics (e.g.,
  mean entropy similarity ± SD) plus per-spectrum score files.
when_to_use_negative:
- When the task is setting up or running new CIDMD collision trajectories — use CIDMD_setup
  instead; this skill only consumes already-generated spectra.
- When only a single molecule/spectrum pair needs qualitative visual comparison —
  the aggregation statistics (mean ± SD, mismatch counting) are designed for a cohort
  of hundreds of spectra.
- When experimental spectra are not from a curated library database like NIST20 —
  score reproducibility against published values (624 ± 189; 828 ± 77) depends on
  that specific reference set.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CIDMD_compare
  role: Computes cosine, mass-weighted dot, and entropy similarity scores between
    CIDMD-predicted spectra and reference spectra from NIST20 and writes similarity_scores.csv
  repo: https://github.com/jesilee/CIDMD_compare
- name: CIDMD_analysis
  role: Generates the in silico mass spectrum files (cidmd.msp, cidmd.jdx) that serve
    as the theoretical inputs for the scoring workflow
  repo: https://github.com/jesilee/CIDMD_analysis
- name: NIST20
  role: Authoritative source of experimental MS/MS library spectra used as the reference
    side of every comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jcim.4c00760
    title: Predicting Collision-Induced-Dissociation Tandem Mass Spectra (CID-MS/MS)
      Using Ab Initio Molecular Dynamics
  source_content_hash: sha256:be0ce91447c9bd6f94e049d4a60e88ed95bf17d1b32251e48c8d29d325c8f3fa
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/jesilee/CIDMD_compare
  tools:
  - CIDMD
  - CIDMD_analysis
  - CIDMD_compare
  - NIST20
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# cidmd-spectrum-aggregation-statistics

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Aggregates similarity scores across all CIDMD-predicted in silico MS/MS spectra by computing cosine, mass-weighted dot, and entropy similarity against NIST20 experimental spectra and reporting cohort-level mean ± standard deviation statistics.

## When to use

Use when you have a cohort of predicted (in silico) MS/MS spectra and matched experimental library spectra and need cohort-level accuracy statistics (e.g., mean entropy similarity ± SD) plus per-spectrum score files.

## When NOT to use

- When the task is setting up or running new CIDMD collision trajectories — use CIDMD_setup instead; this skill only consumes already-generated spectra.
- When only a single molecule/spectrum pair needs qualitative visual comparison — the aggregation statistics (mean ± SD, mismatch counting) are designed for a cohort of hundreds of spectra.
- When experimental spectra are not from a curated library database like NIST20 — score reproducibility against published values (624 ± 189; 828 ± 77) depends on that specific reference set.

## Inputs

- 261 CIDMD-predicted in silico mass spectra in MSP/JDX format (cidmd.msp, cidmd.jdx per molecule)
- corresponding experimental MS/MS spectra retrieved from the NIST20 spectral database
- mapping of each in silico spectrum to its experimental reference spectrum (including molecules with multiple NIST20 reference MS/MS)

## Outputs

- similarity_scores.csv with cosine, mass-weighted dot, and entropy score columns for every spectrum-pair comparison
- cohort-level aggregate statistics: mean entropy similarity ± SD for all spectra and for optimal-parameter (most probable protomer) spectra
- count of complete mismatch spectrum pairs
- PNG visualizations of spectrum comparisons (cidmd.PS_case1_RCID_vfac8-cut05norm.png, cidmd.PS_case1_RCID_vfac8-cut05.png)

## How to apply

Load the 261 generated CIDMD in silico mass spectra (from CIDMD_analysis outputs, e.g., cidmd.msp / cidmd.jdx per molecule) alongside their corresponding experimental MS/MS spectra from NIST20. For every spectrum-pair comparison, compute three similarity scores: cosine (cos) on peak intensities I; mass-weighted dot (Wdot) using W = [PeakIntensity]^0.6 × [Mass]^0.3; and entropy similarity using inverse mass-weighted intensity weighting Iw with w = 1 for S ≥ 3 and w = 0.25 + S × 0.25 for S < 3, combined via 1 − 2^(−S_AB − S_A − S_B)/ln4. Where NIST20 contains multiple MS/MS spectra per molecule, compute all three scores for each reference comparison. Aggregate the entropy similarity scores across all spectra to reproduce the reported cohort average of 624 ± 189 for all spectra and 828 ± 77 for the most probable protomer parameters, and count complete mismatches to confirm the 4-fold reduction. Persist all three score columns per spectrum-pair into a similarity_scores.csv so downstream verification can compare computed aggregates against the published values.

## Related tools

- **CIDMD_compare** (Computes cosine, mass-weighted dot, and entropy similarity scores between CIDMD-predicted spectra and reference spectra from NIST20 and writes similarity_scores.csv) — https://github.com/jesilee/CIDMD_compare
- **CIDMD_analysis** (Generates the in silico mass spectrum files (cidmd.msp, cidmd.jdx) that serve as the theoretical inputs for the scoring workflow) — https://github.com/jesilee/CIDMD_analysis
- **NIST20** (Authoritative source of experimental MS/MS library spectra used as the reference side of every comparison)

## Examples

```
git clone https://github.com/jesilee/CIDMD_compare && cd CIDMD_compare && python compare.py --theoretical ../Molecule/cid/results/cidmd.msp --library nist20.msms --output similarity_scores.csv
```

## Evaluation signals

- Aggregate entropy similarity across all 261 spectra must equal 624 ± 189 (reported value).
- Aggregate entropy similarity for the most probable protomer parameters must equal 828 ± 77.
- similarity_scores.csv must contain all three score columns (cos, Wdot, entropy) for every spectrum-pair comparison, with no missing rows.
- Cosine scores should cluster around 576 ± 349 and Wdot around 572 ± 332, both lower and wider than entropy, providing an internal sanity check on score ordering.
- Count of complete mismatches should show a 4-fold reduction when using optimal (most probable protomer) parameters versus default parameters.

## Limitations

- CIDMD failed to predict the abundance of some fragment ions (e.g., ribono-γ-lactone m/z 69.033) because the required structural precursor was geometrically too far apart, so score aggregation can mask fragment-specific failures.
- Prediction accuracy decreases as the number of protomers and tautomers increases; aggregation over a chemically diverse cohort may understate accuracy for simple molecules and overstate it for complex ones.
- No systematic investigation of radical fragment ions in CID tandem mass spectra has been done; entropy similarity may be skewed by such unmodeled species.
- Discrepancy between CIDMD modeling (simulating multiple angles of collisions and summing many trajectories) and experimental MS/MS conditions may account for part of the mismatch in aggregate scores.

## Evidence
