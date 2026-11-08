---
name: cosine-hungarian-score-configuration
description: Use when pairwise MS2 similarity scores from the Python matchms library
  (e.g. CosineHungarian) must be computed on Spectra objects in R, or when cross-language
  comparison of scoring functions (Cosine vs CosineHungarian) is the analysis goal.
when_to_use_negative:
- When only Spectra's built-in cosine/dot-product scores are needed and no cross-language
  or matchms-specific scoring is required — plain compareSpectra() suffices and avoids
  Python setup overhead
- When the similarity function you need (e.g. ModifiedCosine against an MGF spectral
  library) is part of a metabolite-annotation workflow rather than plain pairwise
  scoring — use the annotation-specific workflow instead
- When data cannot be represented as Spectra/matchms spectrum objects (e.g. already
  aggregated feature tables) — score conversion requires peak-level mz/intensity lists
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SpectriPy
  role: R wrapper that converts Spectra::Spectra to matchms.Spectrum objects and exposes
    matchms scoring functions as the compareSpectriPy() R function
  repo: https://github.com/rformassspectrometry/SpectriPy
- name: Spectra
  role: Provides the Spectra object infrastructure and the built-in compareSpectra()
    reference similarity calculation
  repo: https://github.com/RforMassSpectrometry/Spectra
- name: matchms
  role: Python library supplying the CosineHungarian/CosineGreedy similarity functions
    and the tolerance parameter
  repo: https://github.com/matchms
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:5ab657d3ccc739d66ba2f50607467a5ad8c2d6821595ebf5989ecbef5578977a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rformassspectrometry/SpectriPy
  tools:
  - R
  - Spectra
  - SpectriPy
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cosine-hungarian-score-configuration

## Summary

Configure and run matchms's CosineHungarian (and sibling CosineGreedy) similarity scoring on R Spectra objects via SpectriPy's compareSpectriPy(), producing a numeric pairwise similarity matrix directly comparable to Spectra's built-in compareSpectra() scores.

## When to use

Use when pairwise MS2 similarity scores from the Python matchms library (e.g. CosineHungarian) must be computed on Spectra objects in R, or when cross-language comparison of scoring functions (Cosine vs CosineHungarian) is the analysis goal.

## When NOT to use

- When only Spectra's built-in cosine/dot-product scores are needed and no cross-language or matchms-specific scoring is required — plain compareSpectra() suffices and avoids Python setup overhead
- When the similarity function you need (e.g. ModifiedCosine against an MGF spectral library) is part of a metabolite-annotation workflow rather than plain pairwise scoring — use the annotation-specific workflow instead
- When data cannot be represented as Spectra/matchms spectrum objects (e.g. already aggregated feature tables) — score conversion requires peak-level mz/intensity lists

## Inputs

- Spectra::Spectra object with MS2 spectra (mz and intensity peak lists, precursorMz, msLevel metadata)
- Target Spectra object (e.g. caffeine MS2 spectra) against which similarities are computed
- Scoring parameter object, e.g. CosineHungarian(tolerance = 0.05)

## Outputs

- Numeric matrix of pairwise similarity scores (rows = query spectra, columns = target spectra)
- Optional combined table of Spectra vs matchms scores (e.g. similarity_scores.csv) and a scatter plot (comparison_scatter.png)

## How to apply

Build the Spectra objects in R (e.g. two MS2 spectra for Caffeine with precursorMz 195.0877 and two for 1-Methylhistidine, precursorMz 170.0924, HMDB0000001), combine them with c(caf, mhd), and compute a reference matrix with Spectra's built-in compareSpectra(all, caf). Then call compareSpectriPy(all, caf, param = CosineHungarian(tolerance = 0.05)), setting tolerance to 0.05 rather than the default 0.1 to tighten peak matching during the cosine computation; the same param-object pattern applies to CosineGreedyParam and other matchms scoring functions. SpectriPy converts the Spectra::Spectra objects to matchms.Spectrum objects under the hood via reticulate, runs the Python scoring function, and returns a numeric matrix of similarities with one row per query spectrum and one column per target spectrum. Note that the first compareSpectriPy() call is slower because the Python setup has to be initialized.

## Related tools

- **SpectriPy** (R wrapper that converts Spectra::Spectra to matchms.Spectrum objects and exposes matchms scoring functions as the compareSpectriPy() R function) — https://github.com/rformassspectrometry/SpectriPy
- **Spectra** (Provides the Spectra object infrastructure and the built-in compareSpectra() reference similarity calculation) — https://github.com/RforMassSpectrometry/Spectra
- **matchms** (Python library supplying the CosineHungarian/CosineGreedy similarity functions and the tolerance parameter) — https://github.com/matchms

## Examples

```
res <- compareSpectriPy(all, caf, param = CosineHungarian(tolerance = 0.05))
```

## Evaluation signals

- The returned similarity matrix has the expected dimensions (e.g. 4 rows x 2 columns for two caffeine + two 1-methylhistidine spectra scored against the caffeine spectra)
- All similarity values lie in [0, 1]
- Self-similarity entries (caffeine spectra scored against themselves) equal 1
- compareSpectriPy() returns a numeric matrix like compareSpectra(), allowing direct element-wise comparison across the two scoring routes
- The tolerance parameter is set to 0.05 (not the matchms default 0.1) as specified in the vignette workflow

## Limitations

- The first compareSpectriPy() call in a session is slower because the Python/reticulate setup has to be initialized
- SpectriPy requires Python (version >= 3.12); required Python libraries (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0) must be installed or are auto-installed via reticulate's py_require()
- If RETICULATE_PYTHON or RETICULATE_PYTHON_ENV environment variables are set, all required Python libraries must be installed manually since reticulate will not install them automatically

## Evidence

- [readme] The SpectriPy package provides the compareSpectriPy() function that allows to perform spectra similarity calculations using the scoring functions from MS Python packages.: "The *SpectriPy* package provides the `compareSpectriPy()` function that allows
to perform spectra similarity calculations using the scoring functions from MS
Python packages."
- [readme] Similarity calculated with CosineGreedy from matchms, changing tolerance to 0.05 instead of the default 0.1.: "We calculate the similarity using the *CosineGreedy* function from *matchms*,
changing the `tolerance` to a value of `0.05` (instead of the default `0.1`)."
- [readme] compareSpectriPy() returns a numeric matrix of similarities.: "As a result `compareSpectriPy()` returns also a numeric matrix of similarities."
- [readme] The first compareSpectriPy() call takes longer because the Python setup has to be initialized.: "Note also that the first `compareSpectriPy()` call takes usually a little longer
because the Python setup has to be initialized."
- [abstract] SpectriPy converts between R's Spectra and Python's matchms.Spectrum objects.: [evidence span withheld — anchor status: not_found]
