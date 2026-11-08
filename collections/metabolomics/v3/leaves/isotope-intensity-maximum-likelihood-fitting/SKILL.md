---
name: isotope-intensity-maximum-likelihood-fitting
description: Use when scoring candidate molecular formulas from high-resolution MS1
  isotope peak intensities, or when reimplementing/validating SIRIUS 4's isotope pattern
  scoring with explicit absolute and relative error parameters.
when_to_use_negative:
- No MS1 isotope peak intensities are available (e.g. the spectrum contains only the
  precursor peak); SIRIUS only uses the MS1 parent peak for decomposition when MS1
  data is available
- Mass spectra are not of high resolution, so isotope peaks cannot be resolved and
  ranked isotope patterns cannot be computed
- The goal is MS2-based analysis (fragmentation tree computation, CSI:FingerID/CANOPUS/MSNovelist
  structure search) rather than elemental composition scoring from isotope patterns
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: SIRIUS 4
  role: Source of the intensity error model (Y = x + D + E), the closed-form maximum
    likelihood (equation 2), the x/y swap in the implementation, and the default parameters
    sigma_abs = 0.01 and sigma_rel = 0.08 being reconstructed
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS
  role: Provides the demo dataset (demo-data.zip) used to extract measured MS1 isotope
    peak intensities for validating the reimplemented scoring model
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:18447ecb112246441fbcc4bc9892e2a2c06569b3b372a2b9cff9c9d1a03de990
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - SIRIUS
  - SIRIUS 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# isotope-intensity-maximum-likelihood-fitting

## Summary

Reconstructs SIRIUS 4's isotope pattern intensity scoring model, in which measured isotope peak intensities are scored against theoretical isotope patterns of candidate molecular formulas via a closed-form maximum likelihood for combined absolute and relative Gaussian intensity noise.

## When to use

Use when scoring candidate molecular formulas from high-resolution MS1 isotope peak intensities, or when reimplementing/validating SIRIUS 4's isotope pattern scoring with explicit absolute and relative error parameters.

## When NOT to use

- No MS1 isotope peak intensities are available (e.g. the spectrum contains only the precursor peak); SIRIUS only uses the MS1 parent peak for decomposition when MS1 data is available
- Mass spectra are not of high resolution, so isotope peaks cannot be resolved and ranked isotope patterns cannot be computed
- The goal is MS2-based analysis (fragmentation tree computation, CSI:FingerID/CANOPUS/MSNovelist structure search) rather than elemental composition scoring from isotope patterns

## Inputs

- Measured MS1 isotope peak intensities per compound (list of peak masses and abundances after mass-spectrometer preprocessing), e.g. from the SIRIUS demo-data.zip dataset
- Candidate molecular formulas per compound
- Per-element natural isotope abundances for computing theoretical isotope patterns by combinatorial convolution
- Error model parameters sigma_abs (default 0.01) and sigma_rel (default 0.08)

## Outputs

- Per-candidate-formula total log-odds isotope pattern scores per compound
- Ranked candidate formula list per compound
- isotope_pattern_scores.csv result table

## How to apply

Model the observed intensity as Y = x + D + E, where D ~ N(0, x^2 * sigma_rel^2) is independent relative noise and E ~ N(0, sigma_abs^2) is independent absolute noise; because both terms are Gaussian, the maximum likelihood (equation 2 of Dührkop et al. 2019) has a closed form depending only on y - x, so no numerical optimization is needed. Compute theoretical isotope patterns for each candidate formula by convolving per-element natural isotope abundances, then evaluate the closed-form likelihood analytically for each measured/theoretical intensity pair. Follow SIRIUS 4's implementation detail: swap x and y so relative noise depends on the observed rather than the candidate intensity. Normalize each maximum likelihood to a log-odds score against the noise-only baseline using delta0 = 2*sigma_rel and epsilon0 = 2*sigma_abs, with defaults sigma_abs = 0.01 and sigma_rel = 0.08. Sum log-odds across all isotope peaks of a compound to rank candidate molecular formulas, and write the ranked table to isotope_pattern_scores.csv.

## Related tools

- **SIRIUS 4** (Source of the intensity error model (Y = x + D + E), the closed-form maximum likelihood (equation 2), the x/y swap in the implementation, and the default parameters sigma_abs = 0.01 and sigma_rel = 0.08 being reconstructed) — https://github.com/sirius-ms/sirius
- **SIRIUS** (Provides the demo dataset (demo-data.zip) used to extract measured MS1 isotope peak intensities for validating the reimplemented scoring model) — https://github.com/sirius-ms/sirius

## Examples

```
from scipy.stats import norm; logodds = norm.logpdf(y, loc=x, scale=y*0.08) - norm.logpdf(y, loc=0, scale=2*0.08)
```

## Evaluation signals

- Closed-form implementation numerically reproduces equation (2) of the SIRIUS 4 model on synthetic measured/theoretical intensity pairs
- Every compound in the demo dataset receives finite log-odds scores for all candidate formulas in isotope_pattern_scores.csv
- Log-odds normalization uses delta0 = 2*sigma_rel and epsilon0 = 2*sigma_abs with the defaults sigma_abs = 0.01 and sigma_rel = 0.08
- Relative noise term is computed from the observed intensity y (not the candidate intensity x), matching the SIRIUS implementation's x/y swap
- Total scores are a sum of per-peak log-odds across all isotope peaks, and candidate formulas rank consistently under the noise-only baseline

## Limitations

- The implementation swaps x and y so relative noise depends on the observed rather than the candidate intensity, a deliberate deviation from the idealized model Y = x + D + E
- The Gaussian error model is required for the closed form; the analytic maximum likelihood is only valid under the assumed independent normal absolute and relative noise
- SIRIUS's isotope pattern analysis was rewritten with stability stated only as 'hopefully more stable than before', and earlier releases had bugs in isotope pattern selection and used the old scoring system by default when the -p parameter was not given

## Evidence

- [methods] SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution.: "SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution."
- [methods] After preprocessing, the output of a mass spectrometer is a list of peaks which corresponds to the masses of the sample molecules and their abundance.: "After preprocessing, the output of a mass spectrometer is a list of peaks which corresponds to the masses of the sample molecules and their abundance."
- [readme] SIRIUS is a java-based software framework for the analysis of LC-MS/MS data of metabolites and other "small molecules of biological interest".: [evidence span withheld — anchor status: external_doc]
- [discussion] isotope pattern analysis is now rewritten and hopefully more stable than before: "isotope pattern analysis is now rewritten and hopefully more stable than before"
- [discussion] if MS1 data is available, SIRIUS will now always use the parent peak from MS1 to decompose the: "if MS1 data is available, SIRIUS will now always use the parent peak from MS1 to decompose the"
