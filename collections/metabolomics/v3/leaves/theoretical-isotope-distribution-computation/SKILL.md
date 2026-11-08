---
name: theoretical-isotope-distribution-computation
description: Use when high-resolution MS1 isotope peak lists (peak masses and abundances)
  must be assigned a molecular formula by ranking candidate isotope patterns, or when
  reimplementing SIRIUS 4's isotope pattern intensity scoring.
when_to_use_negative:
- Input is already a ranked formula/structure list from SIRIUS and no intensity-based
  formula re-scoring is needed
- Only MS2 fragmentation spectra are available without MS1 isotope peak data — use
  fragmentation tree computation instead
- Spectra are low-resolution, where SIRIUS's isotope pattern analysis of 'mass spectra
  of high resolution' does not apply
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SIRIUS 4
  role: Reference implementation whose isotope pattern analysis ranks molecular formulas
    by scoring measured MS1 isotope patterns against theoretical ones
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS
  role: Java-based framework providing the isotope pattern analysis workflow step
    and demo data
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:08ceb2b6e61d82d1b0a0b68a864dc8c94d6504ced046535367f4edf1a34c5428
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - SIRIUS
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# theoretical-isotope-distribution-computation

## Summary

Computes theoretical isotope patterns for candidate molecular formulas by convolving per-element natural isotope abundances, and scores measured MS1 isotope peak intensities against them using SIRIUS 4's closed-form maximum-likelihood intensity error model (σ_abs = 0.01, σ_rel = 0.08) to rank formulas by summed log-odds.

## When to use

Use when high-resolution MS1 isotope peak lists (peak masses and abundances) must be assigned a molecular formula by ranking candidate isotope patterns, or when reimplementing SIRIUS 4's isotope pattern intensity scoring.

## When NOT to use

- Input is already a ranked formula/structure list from SIRIUS and no intensity-based formula re-scoring is needed
- Only MS2 fragmentation spectra are available without MS1 isotope peak data — use fragmentation tree computation instead
- Spectra are low-resolution, where SIRIUS's isotope pattern analysis of 'mass spectra of high resolution' does not apply

## Inputs

- Measured MS1 isotope peak intensities per compound (peak masses and abundances after mass spectrometer preprocessing), e.g. from the SIRIUS demo-data.zip dataset
- Candidate molecular formulas per compound
- Per-element natural isotope abundance tables

## Outputs

- Theoretical isotope patterns per candidate molecular formula
- Per-peak log-odds scores against the noise-only baseline
- Ranked candidate molecular formulas per compound with total log-odds scores (e.g. isotope_pattern_scores.csv)

## How to apply

Parse the measured MS1 isotope peak intensities (list of peak masses and abundances) produced by the mass spectrometer after preprocessing, as in the SIRIUS demo-data.zip dataset. For each candidate molecular formula, compute the theoretical isotope pattern by combinatorial convolution of per-element natural isotope abundances. Model the intensity error as Y = x + D + E, with independent relative noise D ~ N(0, x^2·σ_rel^2) and absolute noise E ~ N(0, σ_abs^2); because both terms are Gaussian, evaluate the closed-form maximum likelihood (equation 2 in the paper) analytically for each measured/theoretical intensity pair — no numerical optimization is needed. The implementation swaps x and y so relative noise depends on the observed rather than candidate intensity, then normalizes to a log-odds score against a noise-only baseline using δ0 = 2σ_rel and ε0 = 2σ_abs with defaults σ_abs = 0.01 and σ_rel = 0.08. Sum log-odds across isotope peaks per candidate formula and rank formulas per compound.

## Related tools

- **SIRIUS 4** (Reference implementation whose isotope pattern analysis ranks molecular formulas by scoring measured MS1 isotope patterns against theoretical ones) — https://github.com/sirius-ms/sirius
- **SIRIUS** (Java-based framework providing the isotope pattern analysis workflow step and demo data) — https://github.com/sirius-ms/sirius

## Examples

```
python score_isotope_patterns.py --demo demo-data.zip --sigma-abs 0.01 --sigma-rel 0.08 --output isotope_pattern_scores.csv
```

## Evaluation signals

- Closed-form implementation numerically reproduces equation (2) on synthetic measured/theoretical intensity pairs
- Every compound in the demo dataset receives finite log-odds scores for all candidate formulas in the output table (isotope_pattern_scores.csv)
- Relative-noise variance follows x^2·σ_rel^2 with defaults σ_abs = 0.01 and σ_rel = 0.08; log-odds normalization uses δ0 = 2σ_rel and ε0 = 2σ_abs
- Correct formula for each demo compound ranks first when candidates are sorted by total summed log-odds
- Score depends only on the difference between observed intensity y and expected intensity x, as in equation (2)

## Limitations

- The implementation swaps x and y so relative noise depends on the observed rather than candidate intensity — a deviation from the nominal model that reimplementations must match to reproduce SIRIUS scores
- Isotope pattern analysis was rewritten in later SIRIUS versions and was previously unstable; bug fixes in isotope pattern selection were required
- The theoretical pattern must be computed from the parent peak of MS1 when MS1 data is available, per SIRIUS behavior

## Evidence

- [methods] SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution.: "SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution."
- [readme] After preprocessing, the output of a mass spectrometer is a list of peaks which corresponds to the masses of the sample molecules and their abundance.: "After preprocessing, the output of a mass spectrometer is a list of peaks which corresponds to the masses of the sample molecules and their abundance."
- [readme] Demo data link in SIRIUS README.: "- [Demo data](https://github.com/sirius-ms/sirius/raw/refs/heads/stable/data/demo-data.zip?download=)"
- [discussion] Isotope pattern analysis rewritten for stability.: "isotope pattern analysis is now rewritten and hopefully more stable than before"
