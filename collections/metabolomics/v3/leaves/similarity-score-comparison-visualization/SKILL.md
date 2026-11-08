---
name: similarity-score-comparison-visualization
description: Use when you need to benchmark cross-language spectral similarity scoring
  on the same peak lists — comparing R Spectra cosine/dot-product scores against Python
  matchms scores — or to visualize score agreement per spectrum pair.
when_to_use_negative:
- Your data are already similarity/feature matrices and no raw MS2 peak lists are
  available — the score must be recomputed from spectra
- You need scoring functions unavailable in matchms or Spectra; this skill covers
  the cosine-family scores exposed via CosineGreedyParam/compareSpectriPy
- You need single-language scoring only, where a direct call to compareSpectra() or
  matchms alone suffices without the cross-language benchmark
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Spectra
  role: Provides Spectra object infrastructure and the built-in compareSpectra() function
    used to compute the reference (R-side) similarity matrix
  repo: https://github.com/RforMassSpectrometry/Spectra
- name: SpectriPy
  role: 'Bridges R and Python: provides compareSpectriPy() to apply matchms scoring
    functions (CosineGreedyParam) on R Spectra objects'
  repo: https://github.com/rformassspectrometry/SpectriPy
- name: matchms
  role: Python library whose similarity scoring functions (e.g. CosineGreedy, CosineHungarian,
    tolerance parameter) supply the cross-language scores
  repo: https://github.com/matchms
- name: reticulate
  role: R package used by SpectriPy to initialize Python and translate between R Spectra
    and Python matchms.Spectrum objects
  repo: https://rstudio.github.io/reticulate
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:cf7111bb3b85d74556770ff2759145b417fcf12701704056b9d1e1349983f75e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RforMassSpectrometry/Spectra
  tools:
  - R
  - Spectra
  - SpectriPy
  - matchms
  - reticulate
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# similarity-score-comparison-visualization

## Summary

Computes pairwise MS2 spectra similarity matrices with both R Spectra's built-in compareSpectra() and SpectriPy's compareSpectriPy() using matchms scoring functions (e.g. CosineHungarian/CosineGreedy), then visualizes the cross-language score agreement as a scatter plot.

## When to use

Use when you need to benchmark cross-language spectral similarity scoring on the same peak lists — comparing R Spectra cosine/dot-product scores against Python matchms scores — or to visualize score agreement per spectrum pair.

## When NOT to use

- Your data are already similarity/feature matrices and no raw MS2 peak lists are available — the score must be recomputed from spectra
- You need scoring functions unavailable in matchms or Spectra; this skill covers the cosine-family scores exposed via CosineGreedyParam/compareSpectriPy
- You need single-language scoring only, where a direct call to compareSpectra() or matchms alone suffices without the cross-language benchmark

## Inputs

- Spectra object(s) built from peak lists: m/z and intensity vectors per MS2 spectrum with precursorMz metadata (e.g. Caffeine 195.0877; 1-Methylhistidine 170.0924, HMDB0000001)
- matchms scoring function parameter (e.g. CosineGreedy or CosineHungarian) with tolerance setting (0.05)

## Outputs

- Numeric similarity score matrix (here 4 rows x 2 columns: all spectra vs caffeine spectra) from each scoring path
- Combined CSV table of both score matrices (similarity_scores.csv)
- Scatter plot of built-in Spectra scores vs compareSpectriPy scores per spectrum pair (comparison_scatter.png)

## How to apply

Build Spectra objects from the m/z and intensity peak lists (e.g. two Caffeine MS2 spectra with precursorMz 195.0877 and two 1-Methylhistidine spectra with precursorMz 170.0924, HMDB0000001), then combine them with c(caf, mhd). Compute the reference matrix with Spectra's built-in compareSpectra(all, caf) (here a 4x2 matrix against the caffeine spectra). Compute the same matrix with compareSpectriPy(all, caf, param = CosineGreedy(tolerance = 0.05)) (or CosineHungarian), tightening tolerance from the matchms default 0.1 to 0.05; the first compareSpectriPy() call is slower because the Python setup must be initialized. Write both matrices side by side to CSV and render a scatter plot of built-in Spectra scores versus matchms scores per spectrum pair to assess cross-language agreement.

## Related tools

- **Spectra** (Provides Spectra object infrastructure and the built-in compareSpectra() function used to compute the reference (R-side) similarity matrix) — https://github.com/RforMassSpectrometry/Spectra
- **SpectriPy** (Bridges R and Python: provides compareSpectriPy() to apply matchms scoring functions (CosineGreedyParam) on R Spectra objects) — https://github.com/rformassspectrometry/SpectriPy
- **matchms** (Python library whose similarity scoring functions (e.g. CosineGreedy, CosineHungarian, tolerance parameter) supply the cross-language scores) — https://github.com/matchms
- **reticulate** (R package used by SpectriPy to initialize Python and translate between R Spectra and Python matchms.Spectrum objects) — https://rstudio.github.io/reticulate

## Examples

```
res_r <- compareSpectra(all, caf); res_py <- compareSpectriPy(all, caf, param = CosineGreedy(tolerance = 0.05))
```

## Evaluation signals

- Score matrix has exactly 4 rows and 2 columns (all spectra vs caffeine spectra)
- All similarity values lie in [0, 1]
- Caffeine self-similarity entries equal 1 in both matrices
- compareSpectriPy() returns a numeric matrix of similarities of the same shape as compareSpectra() output
- Tolerance is set explicitly to 0.05 (not the matchms default 0.1) so both paths use comparable matching windows

## Limitations

- The first compareSpectriPy() call is typically slower because the Python setup must be initialized
- Scores from Spectra's built-in cosine and matchms cosine-family functions are not guaranteed identical; differences must be inspected via the scatter plot
- Requires Python >= 3.12 with matchms 0.31, spectrum_utils 0.3.2 and numpy 2.2.0 (or reticulate-managed automatic installation); conflicts with pre-set RETICULATE_PYTHON environments must contain all required libraries
- Tolerance choice (0.05 vs default 0.1) affects peak matching and hence scores

## Evidence

- [readme] The SpectriPy package provides the compareSpectriPy() function that allows to perform spectra similarity calculations using the scoring functions from MS Python packages.: "The *SpectriPy* package provides the `compareSpectriPy()` function that allows
to perform spectra similarity calculations using the scoring functions from MS
Python packages."
- [readme] We calculate the similarity using the CosineGreedy function from matchms, changing the tolerance to a value of 0.05 (instead of the default 0.1).: "We calculate the similarity using the *CosineGreedy* function from *matchms*,
changing the `tolerance` to a value of `0.05` (instead of the default `0.1`)."
- [readme] As a result compareSpectriPy() returns also a numeric matrix of similarities.: "As a result `compareSpectriPy()` returns also a numeric matrix of similarities."
- [abstract] SpectriPy provides functionality to directly apply matchms similarity, filtering and normalization routines on MS data in R.: "includes functionality to directly apply spectral similarity, filtering, normalization, etc. routines from the Python matchms library on MS data in R"
- [readme] The first compareSpectriPy() call takes usually a little longer because the Python setup has to be initialized.: "Note also that the first `compareSpectriPy()` call takes usually a little longer
because the Python setup has to be initialized."
