---
name: matchms-scoring-parameter-configuration
description: Use when computing pairwise MS2 spectral similarity scores in R and you
  need a matchms scoring algorithm other than Spectra's native compareSpectra(), or
  when a scoring parameter such as tolerance must be set (e.g., 0.05 instead of the
  default 0.1).
when_to_use_negative:
- The data are already Python matchms.Spectrum objects and no R/Spectra integration
  is needed — call matchms directly in Python instead
- You need precursor-agnostic plain cosine on spectra lacking valid precursorMz metadata
  (ModifiedCosine/NeutralLossesCosine depend on precursor m/z)
- You only need R-native scoring without matchms functionality — use Spectra's built-in
  compareSpectra() and avoid the Python/reticulate overhead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SpectriPy
  role: Provides compareSpectriPy() and the R parameter wrappers (e.g., CosineGreedyParam)
    that call matchms scoring functions from R
  repo: https://github.com/rformassspectrometry/SpectriPy
- name: matchms
  role: Python library whose similarity scoring functions (CosineGreedy, ModifiedCosine,
    NeutralLossesCosine) execute the actual score computation; tested version 0.31
  repo: https://github.com/matchms
- name: Spectra
  role: R infrastructure holding the MS2 spectra (precursorMz, mz, intensity) and
    providing the baseline compareSpectra() against which matchms scores are compared
  repo: https://github.com/RforMassSpectrometry/Spectra
- name: reticulate
  role: R package that bridges to Python, auto-installs the required Python libraries
    via py_require(), and executes the matchms calls
  repo: https://rstudio.github.io/reticulate
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:907d57a3108f04e30143fef90ca099f2ece9a62eb98b8e6446eb823bff07a4bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rformassspectrometry/SpectriPy
  tools:
  - R
  - Spectra
  - SpectriPy
  - matchms
  - reticulate
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# matchms-scoring-parameter-configuration

## Summary

Configure and invoke matchms spectral similarity scoring functions (CosineGreedy, ModifiedCosine, NeutralLossesCosine) through SpectriPy's compareSpectriPy(), including setting scoring parameters such as the m/z tolerance. This extends Spectra-based R workflows with Python matchms scoring algorithms via the reticulate bridge.

## When to use

Use when computing pairwise MS2 spectral similarity scores in R and you need a matchms scoring algorithm other than Spectra's native compareSpectra(), or when a scoring parameter such as tolerance must be set (e.g., 0.05 instead of the default 0.1).

## When NOT to use

- The data are already Python matchms.Spectrum objects and no R/Spectra integration is needed — call matchms directly in Python instead
- You need precursor-agnostic plain cosine on spectra lacking valid precursorMz metadata (ModifiedCosine/NeutralLossesCosine depend on precursor m/z)
- You only need R-native scoring without matchms functionality — use Spectra's built-in compareSpectra() and avoid the Python/reticulate overhead

## Inputs

- R Spectra::Spectra objects containing MS2 spectra with precursorMz, mz, and intensity metadata (e.g., DataFrame-backed spectra for Caffeine and 1-Methylhistidine)
- A reference Spectra object to score against (e.g., the two caffeine MS2 spectra)
- Scoring parameter object (CosineGreedyParam / ModifiedCosine / NeutralLossesCosine) with chosen tolerance

## Outputs

- Numeric matrix of pairwise spectral similarity scores (queries x references)
- Optionally, a combined table of similarity scores with an annotation column naming the scoring algorithm (e.g., written to similarity_scores.csv)

## How to apply

Load Spectra and SpectriPy, then build Spectra objects with msLevel, precursorMz, mz and intensity metadata (e.g., the caffeine spectra with precursorMz 195.0877 and 1-Methylhistidine with precursorMz 170.0924). Combine query and reference sets with c() and call compareSpectriPy(query, reference, param = <ScoringFunction>(...)), where the param is a matchms scoring function wrapped as an R parameter object such as CosineGreedy(tolerance = 0.05), ModifiedCosine(tolerance = 0.05), or NeutralLossesCosine(tolerance = 0.05). The tolerance parameter controls the allowed m/z deviation for peak matching, so adjust it relative to matchms' default of 0.1 based on mass accuracy; ModifiedCosine and NeutralLossesCosine are precursor-aware and additionally use the precursorMz metadata (NeutralLossesCosine also has an ignore_peaks_above_precursor setting). Under the hood, SpectriPy translates R Spectra objects to Python matchms.Spectrum objects, executes the matchms scoring in Python via reticulate, and converts results back, so the first call incurs Python-setup initialization time. The return value is a numeric similarity matrix of the same shape as Spectra's native compareSpectriPy/compareSpectra output, which can be reshaped into a table with rows per query spectrum, columns per reference spectrum, and a column naming the scoring algorithm.

## Related tools

- **SpectriPy** (Provides compareSpectriPy() and the R parameter wrappers (e.g., CosineGreedyParam) that call matchms scoring functions from R) — https://github.com/rformassspectrometry/SpectriPy
- **matchms** (Python library whose similarity scoring functions (CosineGreedy, ModifiedCosine, NeutralLossesCosine) execute the actual score computation; tested version 0.31) — https://github.com/matchms
- **Spectra** (R infrastructure holding the MS2 spectra (precursorMz, mz, intensity) and providing the baseline compareSpectra() against which matchms scores are compared) — https://github.com/RforMassSpectrometry/Spectra
- **reticulate** (R package that bridges to Python, auto-installs the required Python libraries via py_require(), and executes the matchms calls) — https://rstudio.github.io/reticulate

## Examples

```
res <- compareSpectriPy(all, caf, param = CosineGreedy(tolerance = 0.05))
```

## Evaluation signals

- compareSpectriPy() returns a numeric similarity matrix with the same dimensions (n_query x n_reference) as Spectra's native compareSpectra() on the same inputs
- Scores lie in the valid similarity range of the chosen algorithm (e.g., cosine-family scores between 0 and 1, higher for more similar spectra)
- The cross-language result for CosineGreedy at tolerance = 0.05 is consistent with (same score pattern as) the native compareSpectriPy/compareSpectra baseline on the caffeine/1-Methylhistidine example
- The output table has one row per query spectrum, one column per reference spectrum, and a column naming the scoring algorithm
- The first compareSpectriPy() call takes longer (Python setup initialization) while subsequent calls are faster — a behavioral sanity check that the bridge initialized correctly

## Limitations

- Requires Python >= 3.12 on the system, with matchms 0.31, spectrum_utils 0.2.2/0.3.2 and numpy 2.2.0 as the tested versions; if RETICULATE_PYTHON or RETICULATE_PYTHON_ENV is set, all required Python libraries must be installed manually because reticulate will not auto-install them
- The first compareSpectriPy() call in a session is noticeably slower due to Python setup initialization
- Precursor-aware algorithms (ModifiedCosine, NeutralLossesCosine) require correct precursorMz metadata on the input spectra; missing or wrong precursor m/z invalidates the scores

## Evidence

- [methods] The SpectriPy package provides the compareSpectriPy() function that allows to perform spectra similarity calculations using the scoring functions from MS Python packages.: "The *SpectriPy* package provides the `compareSpectriPy()` function that allows
to perform spectra similarity calculations using the scoring functions from MS
Python packages."
- [readme] We calculate the similarity using the CosineGreedy function from matchms, changing the tolerance to a value of 0.05 (instead of the default 0.1).: "We calculate the similarity using the *CosineGreedy* function from *matchms*, changing the `tolerance` to a value of `0.05` (instead of the default `0.1`)."
- [readme] Note also that the first compareSpectriPy() call takes usually a little longer because the Python setup has to be initialized.: "Note also that the first `compareSpectriPy()` call takes usually a little longer because the Python setup has to be initialized."
- [readme] The SpectriPy package allows integration of Python MS packages into a Spectra-based MS analysis in R.: "The *SpectriPy* package allows integration of Python MS packages into a
[*Spectra*](https://github.com/RforMassSpectrometry/Spectra)-based MS analysis
in R."
- [abstract] SpectriPy can convert between R's Spectra::Spectra and Python's matchms.Spectrum and spectrum_utils.spectrum.MsmsSpectrum objects.: [evidence span withheld — anchor status: not_found]
