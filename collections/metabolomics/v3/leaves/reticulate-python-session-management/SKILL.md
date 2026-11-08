---
name: reticulate-python-session-management
description: Use when an R workflow must configure a Python environment (py_require,
  RETICULATE_PYTHON) or reference a live Python MS object such as a matchms.Spectrum
  from R instead of importing a static copy of the peak data.
when_to_use_negative:
- Data is already fully resident in R — use Spectra's R-native backends (e.g. MsBackendDataFrame/MsBackendMemory)
  rather than a lazy Python reference
- You need a durable, standalone copy of the spectra rather than a live pointer into
  the Python session; the backend translates subsets on-the-fly and holds no data
  itself
- A custom RETICULATE_PYTHON/RETICULATE_PYTHON_ENV environment lacks the required
  libraries — SpectriPy will not auto-install them once those variables are set, so
  loading will fail
edam_operation: http://edamontology.org/operation_2928
tools:
- name: SpectriPy
  role: Provides MsBackendPy, which points a Spectra object at a named Python session
    object and translates requested subsets from Python to R on-the-fly
  repo: github.com/rformassspectrometry/SpectriPy
- name: reticulate
  role: Manages the Python session and library installation via py_require() (reticulate
    >= 1.41) or RETICULATE_PYTHON/RETICULATE_PYTHON_ENV environment variables
  repo: https://rstudio.github.io/reticulate
- name: Spectra
  role: Defines the R backend infrastructure into which MsBackendPy plugs; supplies
    the Spectra container and peaksData access
  repo: github.com/Rformassspectrometry/Spectra
- name: matchms
  role: Holds the Python-side spectrum objects (matchms.Spectrum, version 0.31) that
    the R backend references
  repo: github.com/matchms
- name: spectrum_utils
  role: Required Python library (0.3.2) in the reticulate-managed environment; its
    MsmsSpectrum objects are also convertible via SpectriPy
  repo: github.com/bittremieux-lab/spectrum_utils
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:e4dfcceb9f3bf6dac487da73962f6056be29365a07f24ae841ba34f4931a81a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - R
  - Spectra
  - SpectriPy
  - matchms
  - reticulate
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_2928
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# reticulate-python-session-management

## Summary

Configure and share a persistent Python session between R and Python for mass spectrometry workflows via reticulate and SpectriPy, so that R can reference Python-side MS objects (e.g. matchms.Spectrum) without copying peak data.

## When to use

Use when an R workflow must configure a Python environment (py_require, RETICULATE_PYTHON) or reference a live Python MS object such as a matchms.Spectrum from R instead of importing a static copy of the peak data.

## When NOT to use

- Data is already fully resident in R — use Spectra's R-native backends (e.g. MsBackendDataFrame/MsBackendMemory) rather than a lazy Python reference
- You need a durable, standalone copy of the spectra rather than a live pointer into the Python session; the backend translates subsets on-the-fly and holds no data itself
- A custom RETICULATE_PYTHON/RETICULATE_PYTHON_ENV environment lacks the required libraries — SpectriPy will not auto-install them once those variables are set, so loading will fail

## Inputs

- A Python session object holding MS data, e.g. a matchms.Spectrum created from an MGF file (named 'mgf_py' in the worked example)
- R session with SpectriPy and Spectra attached and a reticulate-managed Python environment (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0)

## Outputs

- An R Spectra::Spectra object backed by MsBackendPy that references the Python object without copying peak data
- On request, R-side two-column (m/z, intensity) peak tables translated from Python, e.g. exported as peaks_data_spectrum1.csv

## How to apply

Ensure Python >= 3.12 is available and declare the required libraries through reticulate's py_require() (introduced in reticulate 1.41) so SpectriPy auto-installs matchms 0.31, spectrum_utils 0.3.2, and numpy 2.2.0. Alternatively, install those libraries manually in a virtualenv/conda environment and point reticulate at it with the RETICULATE_PYTHON or RETICULATE_PYTHON_ENV environment variables — in that case all listed libraries must already be present, since reticulate will not install them. Create the Python object (e.g. a matchms.Spectrum named 'mgf_py') in the shared session, then in R attach Spectra and SpectriPy and instantiate Spectra('mgf_py', source = MsBackendPy()) so the backend holds only a reference to the Python object. Access subsets (e.g. peaksData of s[1]) to trigger on-the-fly translation of m/z and intensity values from Python to R; the first cross-language call also pays Python-initialization cost. Rationale: this lazy-reference design keeps the small memory footprint of Python-side storage while exposing full data access in R.

## Related tools

- **SpectriPy** (Provides MsBackendPy, which points a Spectra object at a named Python session object and translates requested subsets from Python to R on-the-fly) — github.com/rformassspectrometry/SpectriPy
- **reticulate** (Manages the Python session and library installation via py_require() (reticulate >= 1.41) or RETICULATE_PYTHON/RETICULATE_PYTHON_ENV environment variables) — https://rstudio.github.io/reticulate
- **Spectra** (Defines the R backend infrastructure into which MsBackendPy plugs; supplies the Spectra container and peaksData access) — github.com/Rformassspectrometry/Spectra
- **matchms** (Holds the Python-side spectrum objects (matchms.Spectrum, version 0.31) that the R backend references) — github.com/matchms
- **spectrum_utils** (Required Python library (0.3.2) in the reticulate-managed environment; its MsmsSpectrum objects are also convertible via SpectriPy) — github.com/bittremieux-lab/spectrum_utils

## Examples

```
library(Spectra); library(SpectriPy); s <- Spectra("mgf_py", source = MsBackendPy()); p <- peaksData(s[1]); write.csv(p, "peaks_data_spectrum1.csv", row.names = FALSE)
```

## Evaluation signals

- The exported peaks_data_spectrum1.csv contains exactly the peak count held by the 'mgf_py' object in the Python session
- m/z and intensity values in the CSV match the Python-side spectrum values exactly (no translation drift)
- Only the requested subset (e.g. spectrum [1]) was translated — the backend held no data before peaksData was called
- First cross-language call (e.g. compareSpectriPy or peaksData) takes noticeably longer because the Python setup has to be initialized
- Session starts without error under Python >= 3.12 with reticulate >= 1.41 and the pinned library versions

## Limitations

- Python >= 3.12 is required on the system
- If RETICULATE_PYTHON or RETICULATE_PYTHON_ENV is defined, all required libraries (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0) must be installed manually; reticulate will not install them automatically
- Only the suggested and tested library versions are validated; other matchms/numpy versions are untested
- The on-the-fly translation cost is paid per request, so repeatedly accessing large subsets from Python incurs repeated conversion overhead

## Evidence

- [readme] reticulate auto-install of Python libraries: "All necessary Python libraries (listed below) are automatically installed by the [*reticulate*](https://rstudio.github.io/reticulate) R package."
- [readme] py_require-based library management: "*SpectriPy*'s Python library management uses the [`py_require()`](https://rstudio.github.io/reticulate/reference/py_require.html) function introduced in *reticulate* version 1.41"
- [readme] manual environment must be complete: "If any of these environment variables are defined, all Python libraries listed below **must** be installed, since *SpectriPy* (respectively *reticulate*) will not try to install them"
- [methods] reticulate as cross-language execution bridge: "create custom functions or workflows on `Spectra` objects in Python and executing them in R using the *reticulate* R package, and vice versa"
- [abstract] R-Python data structure translation: "provides functionality to efficiently translate between R and Python MS data structures"
