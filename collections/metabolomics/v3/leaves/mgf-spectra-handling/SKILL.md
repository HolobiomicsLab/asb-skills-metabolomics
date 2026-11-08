---
name: mgf-spectra-handling
description: Use when MS2 peak data lives as a Python object (e.g., a matchms.Spectrum
  loaded from MGF) but the analysis runs in R with Spectra, and you need lazy, on-demand
  access to peaks without duplicating data across languages.
when_to_use_negative:
- The spectra are already fully available as an R Spectra object in memory (e.g.,
  MsBackendDataFrame/MsBackendMemory) — lazy Python bridging adds reticulate overhead
  with no benefit
- The workflow is purely Python-side; matchms can access the MGF Spectrum directly
  without any R/SpectriPy involvement
- Custom RETICULATE_PYTHON or RETICULATE_PYTHON_ENV is set but the required Python
  libraries (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0) are not installed —
  reticulate will not auto-install them and loading will fail
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SpectriPy
  role: Provides the MsBackendPy backend class and R/Python translation layer so an
    R Spectra object points at a Python session object
  repo: https://github.com/rformassspectrometry/SpectriPy
- name: Spectra
  role: R infrastructure whose backend system (MsBackendPy) supplies lazy, on-the-fly
    peak data access
  repo: https://github.com/RforMassSpectrometry/Spectra
- name: matchms
  role: Python library holding the MGF-derived Spectrum object ('mgf_py') in the Python
    session
  repo: https://github.com/matchms
- name: reticulate
  role: R package managing the Python environment (py_require) and the R/Python session
    bridge
  repo: https://rstudio.github.io/reticulate
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:f9423663379b6c4fc7817fc6553c048136910a698eb0c49f02517272f14dc3a5
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# mgf-spectra-handling

## Summary

Handles MGF-derived MS2 spectra across R and Python without copying peak data, using SpectriPy's MsBackendPy backend so an R Spectra::Spectra object references a Python-side matchms.Spectrum object and translates m/z and intensity values on-the-fly only when subsets are requested.

## When to use

Use when MS2 peak data lives as a Python object (e.g., a matchms.Spectrum loaded from MGF) but the analysis runs in R with Spectra, and you need lazy, on-demand access to peaks without duplicating data across languages.

## When NOT to use

- The spectra are already fully available as an R Spectra object in memory (e.g., MsBackendDataFrame/MsBackendMemory) — lazy Python bridging adds reticulate overhead with no benefit
- The workflow is purely Python-side; matchms can access the MGF Spectrum directly without any R/SpectriPy involvement
- Custom RETICULATE_PYTHON or RETICULATE_PYTHON_ENV is set but the required Python libraries (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0) are not installed — reticulate will not auto-install them and loading will fail

## Inputs

- Python-side matchms.Spectrum object (e.g., 'mgf_py') held in the reticulate-managed Python session, typically loaded from MGF data
- R session with Spectra and SpectriPy installed via BiocManager
- reticulate >= 1.41 with py_require()-managed libraries: matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0 (Python >= 3.12)

## Outputs

- R Spectra object backed by MsBackendPy that references (not copies) the Python object
- Two-column (m/z, intensity) peak matrices for requested spectra subsets, translated on-the-fly from Python to R
- Optionally, exported peak tables (e.g., peaks_data_spectrum1.csv) for validation

## How to apply

Configure the Python environment through reticulate's py_require() so the tested library versions are available (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0, Python >= 3.12), then create the matchms.Spectrum object (e.g., 'mgf_py') in the Python session. In R, attach Spectra and SpectriPy and instantiate Spectra('mgf_py', source = MsBackendPy()) so the backend stores only a reference to the Python-side object rather than a copy of the peak arrays. Request subsets (e.g., sps[1] followed by peaksData) to trigger the backend's on-the-fly translation of m/z and intensity values from Python to R for exactly the requested spectra; the first Python-touching call is slower because reticulate initializes the Python setup. Validate by comparing the returned two-column (m/z, intensity) peak table — or an exported CSV — against the peak count and values held by the Python object, confirming no unwanted bulk translation occurred.

## Related tools

- **SpectriPy** (Provides the MsBackendPy backend class and R/Python translation layer so an R Spectra object points at a Python session object) — https://github.com/rformassspectrometry/SpectriPy
- **Spectra** (R infrastructure whose backend system (MsBackendPy) supplies lazy, on-the-fly peak data access) — https://github.com/RforMassSpectrometry/Spectra
- **matchms** (Python library holding the MGF-derived Spectrum object ('mgf_py') in the Python session) — https://github.com/matchms
- **reticulate** (R package managing the Python environment (py_require) and the R/Python session bridge) — https://rstudio.github.io/reticulate

## Examples

```
library(Spectra); library(SpectriPy); sps <- Spectra("mgf_py", source = MsBackendPy()); peaksData(sps[1])
```

## Evaluation signals

- The exported peak CSV for a subset spectrum (e.g., spectrum 1) contains exactly the peak count and m/z/intensity values held by the 'mgf_py' object in the Python session
- Only the requested subset is translated: accessing sps[1] fetches one spectrum's peak table, not the full collection
- No peak data is copied at backend construction — Spectra('mgf_py', source = MsBackendPy()) completes without transferring arrays
- The Python environment resolves to the tested versions: matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0, Python >= 3.12
- The first compareSpectriPy/backend call may take longer due to Python setup initialization; subsequent calls should be fast — a persistent slowdown indicates a configuration problem

## Limitations

- Requires Python >= 3.12 and reticulate >= 1.41 (py_require()-based library management); older reticulate versions lack py_require()
- If RETICULATE_PYTHON or RETICULATE_PYTHON_ENV environment variables are defined, all required Python libraries must be installed manually — reticulate will not install them automatically
- The first Python-touching operation in a session is slower because the Python setup has to be initialized
- Data is translated on-the-fly per request, so repeatedly accessing all spectra individually incurs repeated cross-language translation costs

## Evidence

- [results] All data from the referenced Python data object is accessible in R, with the entire data or subsets of it translated on-the-fly from Python to R only upon request.: [evidence span withheld — anchor status: not_found]
- [readme] Spectra backends retrieve data lazily: [evidence span withheld — anchor status: external_doc]
- [readme] reticulate py_require library versions: "- [*matchms*](https://github.com/matchms) 0.31
- [*spectrum_utils*](https://github.com/bittremieux-lab/spectrum_utils) 0.3.2
- *numpy* 2.2.0"
- [readme] installation requires Python >= 3.12 and reticulate >= 1.41: "*SpectriPy* needs Python (version >= 3.12) to be installed on the system. All necessary Python libraries (listed below) are automatically installed by the"
- [readme] first call slower due to Python setup initialization: "Note also that the first `compareSpectriPy()` call takes usually a little longer because the Python setup has to be initialized."
