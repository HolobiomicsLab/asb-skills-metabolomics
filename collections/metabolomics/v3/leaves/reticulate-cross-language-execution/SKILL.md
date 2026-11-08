---
name: reticulate-cross-language-execution
description: Use when an R-based Spectra workflow needs Python-only MS routines (matchms
  similarity scores, filters like normalize_intensities), or when a Python routine
  must operate on R MS data without exporting files.
when_to_use_negative:
- When equivalent functionality exists natively in R (e.g., Spectra's built-in compareSpectra()),
  since a native call avoids Python initialization overhead
- When the input is not an MS spectra object (e.g., already a feature table or similarity
  matrix) — the conversion layer translates MS peak data, not derived tables
- When reticulate-managed Python >= 3.12 with the pinned libraries (matchms 0.31,
  spectrum_utils 0.3.2, numpy 2.2.0) cannot be provisioned and manual installation
  via RETICULATE_PYTHON is not possible
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: reticulate
  role: R package that initializes the Python environment, auto-installs required
    Python libraries via py_require(), and executes Python functions from R
  repo: https://rstudio.github.io/reticulate
- name: SpectriPy
  role: Provides rspec_to_pyspec()/pyspec_to_rspec() conversion and the reticulate-backed
    wrappers filterSpectriPy() and compareSpectriPy()
  repo: https://github.com/rformassspectrometry/SpectriPy
- name: Spectra
  role: Defines the R Spectra::Spectra object that is translated to/from Python spectrum
    objects
  repo: https://github.com/RforMassSpectrometry/Spectra
- name: matchms
  role: Python library whose filters (normalize_intensities, select_by_mz, etc.) and
    similarity scores (CosineGreedy, ModifiedCosine) are executed on the converted
    spectra
  repo: https://github.com/matchms
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:b20ba1f024b902493811451f2a144fd9c8f3cfb2f7af7c53eda74c5845f9dbdd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Spectra
  - SpectriPy
  - matchms
  - reticulate
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8260869565217391
n_claims: 23
---

# reticulate-cross-language-execution

## Summary

Execute Python mass spectrometry functionality (e.g., matchms filtering and scoring) directly on R Spectra::Spectra data — or custom Python workflows on R data — via the reticulate R package, as implemented in SpectriPy's rspec_to_pyspec/pyspec_to_rspec conversion layer.

## When to use

Use when an R-based Spectra workflow needs Python-only MS routines (matchms similarity scores, filters like normalize_intensities), or when a Python routine must operate on R MS data without exporting files.

## When NOT to use

- When equivalent functionality exists natively in R (e.g., Spectra's built-in compareSpectra()), since a native call avoids Python initialization overhead
- When the input is not an MS spectra object (e.g., already a feature table or similarity matrix) — the conversion layer translates MS peak data, not derived tables
- When reticulate-managed Python >= 3.12 with the pinned libraries (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0) cannot be provisioned and manual installation via RETICULATE_PYTHON is not possible

## Inputs

- R Spectra::Spectra object (e.g., MS2 spectra for caffeine and 1-methylhistidine with mz/intensity peak lists and metadata msLevel, precursorMz, name, id)
- Python matchms.Spectrum or spectrum_utils.spectrum.MsmsSpectrum objects
- spectraVariableMapping / defaultSpectraVariableMapping definitions

## Outputs

- List of converted Python matchms.Spectrum objects (or R Spectra::Spectra on the reverse direction)
- Filtered/scored spectra: numeric similarity matrix from compareSpectriPy(), normalized Spectra from filterSpectriPy()
- Optional pickle of Python spectra and CSV of the applied variable mapping

## How to apply

Configure Python >= 3.12 through reticulate (SpectriPy uses reticulate's py_require() for automatic library installation; matchms 0.31, spectrum_utils 0.3.2 and numpy 2.2.0 are the tested versions). Convert the R Spectra::Spectra object to Python objects with rspec_to_pyspec(pythonLibrary = 'matchms'), which renames/reformats metadata (msLevel, precursorMz, retention time, compound name/id) following the spectraVariableMapping definitions to matchms metadata conventions. Execute the Python function (e.g., a matchms filter wrapped by filterSpectriPy or a scoring function via compareSpectriPy) on the converted objects, then bring results back with pyspec_to_rspec() using the default spectraVariableMapping. Alternatively, wrap arbitrary custom Python functions on Spectra objects and call them from R via reticulate, and vice versa. Note that the first Python call incurs interpreter setup time.

## Related tools

- **reticulate** (R package that initializes the Python environment, auto-installs required Python libraries via py_require(), and executes Python functions from R) — https://rstudio.github.io/reticulate
- **SpectriPy** (Provides rspec_to_pyspec()/pyspec_to_rspec() conversion and the reticulate-backed wrappers filterSpectriPy() and compareSpectriPy()) — https://github.com/rformassspectrometry/SpectriPy
- **Spectra** (Defines the R Spectra::Spectra object that is translated to/from Python spectrum objects) — https://github.com/RforMassSpectrometry/Spectra
- **matchms** (Python library whose filters (normalize_intensities, select_by_mz, etc.) and similarity scores (CosineGreedy, ModifiedCosine) are executed on the converted spectra) — https://github.com/matchms

## Examples

```
library(Spectra); library(SpectriPy); res <- compareSpectriPy(all, caf, param = CosineGreedy(tolerance = 0.05))
```

## Evaluation signals

- Round-trip check: after rspec_to_pyspec() and reload, each matchms.Spectrum reproduces the source spectra's peak m/z and intensity arrays exactly
- Metadata mapping check: every mapped field (MS level, precursor m/z, retention time, name, id) is present under its renamed matchms metadata key per spectraVariableMapping
- Filter behavior check: after filterSpectriPy(normalize_intensities), each output spectrum's intensity vector has maximum equal to 1
- Scoring check: compareSpectriPy() returns a numeric similarity matrix of the expected dimensions, consistent with Spectra's native compareSpectra() on identical inputs
- Environment check: reticulate resolves Python >= 3.12 with matchms 0.31, spectrum_utils 0.3.2 and numpy 2.2.0 available before the first call

## Limitations

- The first reticulate/Python call is slow because the Python setup has to be initialized
- If RETICULATE_PYTHON or RETICULATE_PYTHON_ENV is set, all required Python libraries must be manually installed — reticulate will not install them automatically
- SpectriPy requires Python >= 3.12 and depends on reticulate >= 1.41 for py_require()-based library management; installation or loading can fail and needs the documented fix procedures

## Evidence

- [intro] SpectriPy enables integration of Python mass spectrometry packages into Spectra-based MS workflows via reticulate: "The *SpectriPy* package allows integration of Python MS packages into a
[*Spectra*](https://github.com/RforMassSpectrometry/Spectra)-based MS analysis"
- [methods] Custom Python functions on Spectra objects executed in R via reticulate: "create custom functions or workflows on `Spectra` objects in Python
and executing them in R using the *reticulate* R package, and vice versa"
- [readme] reticulate automatically installs the Python libraries required by SpectriPy: "All
necessary Python libraries (listed below) are automatically installed by the
[*reticulate*](https://rstudio.github.io/reticulate) R package."
- [other] filterSpectriPy translates MS data to Python, runs matchms filtering, and converts results back to R: [evidence span withheld — anchor status: not_found]
