---
name: log-odds-score-normalization
description: Use when ranking candidate molecular formulas from high-resolution MS1
  isotope patterns, where measured peak intensities must be compared to theoretical
  patterns with both absolute (σ_abs = 0.01) and relative (σ_rel = 0.08) noise terms.
when_to_use_negative:
- Input is already a fragmentation tree or MS2 tandem spectrum — this skill scores
  MS1 isotope patterns only, not the fragmentation tree computation step.
- Measured peak intensities are absent (MS1 data not available); the intensity error
  model cannot be evaluated without abundances.
- Candidate molecular formulas are not yet enumerated — this skill ranks formulas,
  it does not generate them.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS 4
  role: Reference implementation of the isotope pattern scoring model; its closed-form
    maximum likelihood and log-odds normalization (equation 2) with defaults σ_abs
    = 0.01 and σ_rel = 0.08 are reproduced here.
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:a0b055060372ee9d6f5d8479451f4b31d2f2f6337aa8acbf7aeb46c1946935e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - SIRIUS
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# log-odds-score-normalization

## Summary

Scores candidate molecular formulas by evaluating a Gaussian intensity error model (relative and absolute noise) in closed-form maximum likelihood and normalizing to a log-odds score summed across isotope peaks.

## When to use

Use when ranking candidate molecular formulas from high-resolution MS1 isotope patterns, where measured peak intensities must be compared to theoretical patterns with both absolute (σ_abs = 0.01) and relative (σ_rel = 0.08) noise terms.

## When NOT to use

- Input is already a fragmentation tree or MS2 tandem spectrum — this skill scores MS1 isotope patterns only, not the fragmentation tree computation step.
- Measured peak intensities are absent (MS1 data not available); the intensity error model cannot be evaluated without abundances.
- Candidate molecular formulas are not yet enumerated — this skill ranks formulas, it does not generate them.

## Inputs

- List of measured MS1 isotope peak masses and abundances from a preprocessed mass spectrum (e.g. from the SIRIUS demo-data.zip dataset)
- Theoretical isotope pattern peak intensities for each candidate molecular formula

## Outputs

- Log-odds score per candidate formula, summed across isotope peaks
- Ranked list of candidate molecular formulas per compound by total log-odds score (e.g. isotope_pattern_scores.csv)

## How to apply

Compute theoretical isotope patterns for each candidate formula by convolving per-element natural isotope abundances, then evaluate the closed-form maximum likelihood of the intensity error model Y = x + D + E, where relative noise D ~ N(0, x²·σ_rel²) and absolute noise E ~ N(0, σ_abs²) are independent. Following the SIRIUS 4 implementation, swap observed and expected intensities so relative noise depends on the measured value y rather than the candidate intensity x, then normalize the maximum likelihood to log odds against a noise-only baseline using δ0 = 2σ_rel and ε0 = 2σ_abs. Sum log-odds across all isotope peaks per candidate formula and rank candidates by the total score. The defaults σ_abs = 0.01 and σ_rel = 0.08 should be used unless recalibrated for a specific instrument.

## Related tools

- **SIRIUS 4** (Reference implementation of the isotope pattern scoring model; its closed-form maximum likelihood and log-odds normalization (equation 2) with defaults σ_abs = 0.01 and σ_rel = 0.08 are reproduced here.) — https://github.com/sirius-ms/sirius

## Examples

```
python score_isotopes.py --input peaks.csv --candidates formulas.csv --sigma_abs 0.01 --sigma_rel 0.08 --output isotope_pattern_scores.csv
```

## Evaluation signals

- Verify the closed-form implementation numerically reproduces equation (2) on synthetic (x, y) intensity pairs before applying to real data.
- Confirm every compound in the dataset receives finite log-odds scores for all candidates in the output table (no NaN/inf).
- Check that the sum of log-odds across isotope peaks produces a single scalar per candidate formula and that rankings are consistent when peak order is permuted.
- Confirm relative noise is computed from the observed intensity y (post-swap), not from the candidate intensity x.

## Limitations

- The implementation swaps x and y so relative noise depends on the observed rather than candidate intensity — this is a modeling choice that differs from the exact generative model.
- Isotope pattern analysis was rewritten in later SIRIUS versions and was previously unstable ('hopefully more stable than before').
- Scoring depends on accurate preprocessing; the mass spectrometer output must already be a list of peak masses and abundances.

## Evidence

- [methods] closed-form maximum likelihood depends only on the difference between observed intensity y and expected intensity x: [evidence span withheld — anchor status: not_found]
- [methods] default parameters σ_abs and σ_rel: [evidence span withheld — anchor status: not_found]
- [methods] swap of observed and expected intensities: [evidence span withheld — anchor status: not_found]
- [methods] isotope pattern ranking from high-resolution spectra: "SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution."
- [readme] preprocessed peak list input: "After preprocessing, the output of a mass spectrometer is a list of peaks which corresponds to the masses of the sample molecules and their abundance."
