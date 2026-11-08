---
name: isotope-pattern-scoring
description: Use when you have high-resolution MS1 isotope peak lists (masses and
  abundances) and need to rank candidate molecular formulas by how well their theoretical
  isotope patterns explain the observed intensities.
when_to_use_negative:
- The input is tandem MS/MS fragmentation spectra rather than MS1 isotope peak intensities
  — for that, fragmentation tree computation is the relevant SIRIUS workflow step,
  not isotope pattern scoring.
- The goal is structure or database search (CSI:FingerID, CANOPUS, MSNovelist) rather
  than ranking elemental compositions of the parent ion.
- Spectra are low-resolution or lack resolvable isotope peaks, so no reliable isotope
  peak intensities can be extracted after preprocessing.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS 4
  role: 'Source of the isotope pattern analysis method being reconstructed: intensity
    error model, closed-form maximum likelihood, log-odds normalization, and default
    scoring parameters'
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS
  role: Java-based framework whose Isotope Pattern Analysis module deduces molecular
    formulas by ranking isotope patterns from high-resolution mass spectra
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:0af88aee71e6b3fcc2d6e4c6f393b233962b00692e5ad7a00fef40778c3d58ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - SIRIUS
  - SIRIUS 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# isotope-pattern-scoring

## Summary

Reproduces SIRIUS 4's maximum-likelihood isotope pattern intensity scoring model, which scores candidate molecular formulas against measured high-resolution MS1 isotope peak intensities using a combined relative+absolute Gaussian noise model with defaults sigma_abs = 0.01 and sigma_rel = 0.08.

## When to use

Use when you have high-resolution MS1 isotope peak lists (masses and abundances) and need to rank candidate molecular formulas by how well their theoretical isotope patterns explain the observed intensities.

## When NOT to use

- The input is tandem MS/MS fragmentation spectra rather than MS1 isotope peak intensities — for that, fragmentation tree computation is the relevant SIRIUS workflow step, not isotope pattern scoring.
- The goal is structure or database search (CSI:FingerID, CANOPUS, MSNovelist) rather than ranking elemental compositions of the parent ion.
- Spectra are low-resolution or lack resolvable isotope peaks, so no reliable isotope peak intensities can be extracted after preprocessing.

## Inputs

- High-resolution MS1 isotope peak list per compound (peak masses and abundances), e.g. extracted from SIRIUS demo-data.zip after preprocessing
- Candidate molecular formulas per compound
- Theoretical isotope patterns computed from per-element natural isotope abundances
- Scoring parameters sigma_abs = 0.01 and sigma_rel = 0.08 (SIRIUS 4 defaults)

## Outputs

- Per-candidate-formula summed log-odds isotope pattern scores
- Ranked candidate molecular formulas per compound
- isotope_pattern_scores.csv with per-compound candidate scores

## How to apply

First parse the input dataset (e.g. the SIRIUS demo-data.zip) and extract, per compound, the measured MS1 isotope peak masses and abundances produced by the mass spectrometer after preprocessing. Compute theoretical isotope patterns for each candidate formula by convolving per-element natural isotope abundances (combinatorial isotope distribution). Model the measured intensity as Y = x + D + E, with independent relative noise D ~ N(0, x^2 * sigma_rel^2) and independent absolute noise E ~ N(0, sigma_abs^2), and evaluate the closed-form maximum likelihood of equation (2) analytically per measured/theoretical intensity pair — no numerical optimization. Note that SIRIUS's implementation swaps x and y so the relative noise variance depends on the observed rather than the candidate intensity. Normalize each maximum likelihood to a log-odds score against the noise-only baseline using delta0 = 2*sigma_rel and epsilon0 = 2*sigma_abs with defaults sigma_abs = 0.01 and sigma_rel = 0.08, sum log-odds across isotope peaks per candidate formula, and rank candidates by total score.

## Related tools

- **SIRIUS 4** (Source of the isotope pattern analysis method being reconstructed: intensity error model, closed-form maximum likelihood, log-odds normalization, and default scoring parameters) — https://github.com/sirius-ms/sirius
- **SIRIUS** (Java-based framework whose Isotope Pattern Analysis module deduces molecular formulas by ranking isotope patterns from high-resolution mass spectra) — https://github.com/sirius-ms/sirius

## Evaluation signals

- Closed-form implementation numerically reproduces equation (2) of the SIRIUS 4 isotope scoring model on synthetic measured/theoretical intensity pairs
- Every compound in the demo dataset receives finite log-odds scores for all candidates in isotope_pattern_scores.csv
- Default parameters sigma_abs = 0.01 and sigma_rel = 0.08 (log-odds normalizers delta0 = 2*sigma_rel, epsilon0 = 2*sigma_abs) are used, matching SIRIUS 4
- The x/y swap is applied so relative noise depends on the observed intensity, not the candidate intensity, matching the SIRIUS implementation
- Ranking of candidates per compound is consistent with the total summed log-odds scores in the output table

## Limitations

- The implementation swaps the expected and observed intensities relative to the theoretical model, so relative noise depends on the observed intensity — a faithful reimplementation must replicate this choice to match SIRIUS scores
- Isotope pattern analysis in SIRIUS was rewritten with stability described only as 'hopefully more stable than before', and there were fixed bugs in isotope pattern selection, so behavior may vary across SIRIUS versions
- Requires MS1 isotope information: if MS1 data is available SIRIUS uses the parent peak from MS1 to decompose the parent ion, so scoring degrades without high-resolution MS1 isotope peaks

## Evidence

- [methods] Isotope Pattern Analysis deduces formulas by ranking isotope patterns: "SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution."
- [readme] Preprocessed mass spectrometer output is a peak list of masses and abundances: "After preprocessing, the output of a mass spectrometer is a list of peaks which corresponds to the masses of the sample molecules and their abundance."
- [discussion] Isotope pattern analysis was rewritten and stability is not guaranteed: "isotope pattern analysis is now rewritten and hopefully more stable than before"
