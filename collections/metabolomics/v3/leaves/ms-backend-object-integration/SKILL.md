---
name: ms-backend-object-integration
description: Use when an R analysis must read peak (m/z, intensity) data that lives
  as a Python object in the shared reticulate session, and copying the full dataset
  into R should be avoided.
when_to_use_negative:
- The data should be fully converted and owned in R (use SpectriPy's Spectra <-> matchms.Spectrum
  conversion functions instead of the lazy backend).
- A local/conda/virtualenv Python is configured via RETICULATE_PYTHON or RETICULATE_PYTHON_ENV
  without all required libraries (matchms, spectrum_utils, numpy) installed, since
  reticulate will not auto-install them.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SpectriPy
  role: Provides the MsBackendPy backend class that points a Spectra object at a Python-side
    data object and translates requested subsets on-the-fly
  repo: https://github.com/rformassspectrometry/SpectriPy
- name: Spectra
  role: R infrastructure whose backend mechanism MsBackendPy plugs into; defines on-the-fly
    data access semantics
  repo: https://github.com/Rformassspectrometry/Spectra
- name: matchms
  role: Python library whose Spectrum objects hold the MS peak data accessed through
    the backend (tested version 0.31)
  repo: https://github.com/matchms
- name: reticulate
  role: Manages the shared R/Python session and py_require()-based installation of
    the required Python libraries
  repo: https://rstudio.github.io/reticulate
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:1b5576bbb038d1cf7706df46aa4a9275f2d6e860efdf5ba015d77a6441993bef
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
yes_rate: 0.6666666666666666
n_claims: 15
---

# ms-backend-object-integration

## Summary

Access MS peak data held as a Python object (e.g., a matchms.Spectrum) from an R Spectra::Spectra object via SpectriPy's MsBackendPy, which translates data from Python to R on-the-fly only when subsets are requested, without copying.

## When to use

Use when an R analysis must read peak (m/z, intensity) data that lives as a Python object in the shared reticulate session, and copying the full dataset into R should be avoided.

## When NOT to use

- The data should be fully converted and owned in R (use SpectriPy's Spectra <-> matchms.Spectrum conversion functions instead of the lazy backend).
- A local/conda/virtualenv Python is configured via RETICULATE_PYTHON or RETICULATE_PYTHON_ENV without all required libraries (matchms, spectrum_utils, numpy) installed, since reticulate will not auto-install them.

## Inputs

- Python object in the reticulate session holding MS data (e.g., matchms.Spectrum named 'mgf_py')
- R session with Spectra and SpectriPy installed via BiocManager
- reticulate-managed Python environment with matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0

## Outputs

- R Spectra::Spectra object backed by MsBackendPy referencing the Python object
- Two-column (m/z, intensity) peak matrix/table for requested spectra subsets, e.g. exported to peaks_data_spectrum1.csv

## How to apply

Configure the Python environment through reticulate (py_require-managed matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0; Python >= 3.12) and create the Python-side data object, e.g. a matchms.Spectrum named 'mgf_py'. In R, attach Spectra and SpectriPy and instantiate Spectra('mgf_py', source = MsBackendPy()) so the backend points at the Python object by name rather than importing its peak data. Subsequent accessors (e.g., peaksData on a subset such as [1]) trigger on-the-fly translation of only the requested m/z/intensity subsets from Python to R. Because data is translated lazily, avoid wholesale extraction when only a few spectra are needed; the backend behaves like other Spectra on-the-fly backends (e.g., MsBackendCompDb) with a small memory footprint in R.

## Related tools

- **SpectriPy** (Provides the MsBackendPy backend class that points a Spectra object at a Python-side data object and translates requested subsets on-the-fly) — https://github.com/rformassspectrometry/SpectriPy
- **Spectra** (R infrastructure whose backend mechanism MsBackendPy plugs into; defines on-the-fly data access semantics) — https://github.com/Rformassspectrometry/Spectra
- **matchms** (Python library whose Spectrum objects hold the MS peak data accessed through the backend (tested version 0.31)) — https://github.com/matchms
- **reticulate** (Manages the shared R/Python session and py_require()-based installation of the required Python libraries) — https://rstudio.github.io/reticulate

## Examples

```
library(Spectra); library(SpectriPy); s <- Spectra('mgf_py', source = MsBackendPy()); peaksData(s[1])
```

## Evaluation signals

- The exported peak table (e.g., peaks_data_spectrum1.csv) contains exactly the peak count and m/z/intensity values held by the 'mgf_py' object in the Python session.
- Only the requested spectrum subset was translated — accessing [1] yields one spectrum's peaks without pulling the full collection into R.
- Spectra('mgf_py', source = MsBackendPy()) initializes without copying peak data; the R object resolves the Python object by name.
- compareSpectriPy()-style calls on the backend-backed object agree with results computed directly in Python.
- The reticulate environment reports the required library versions (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0).

## Limitations

- Data translation from Python to R happens on-the-fly per request, so repeated access to large subsets incurs repeated translation cost.
- The first SpectriPy operation is slower because the Python setup must be initialized.
- If RETICULATE_PYTHON or RETICULATE_PYTHON_ENV is set, all required Python libraries must be pre-installed; reticulate will not install them automatically.
- Requires Python >= 3.12 and reticulate >= 1.41 for py_require()-based library management.

## Evidence

- [results] All data from the referenced Python data object is accessible in R, with the entire data or subsets of it translated on-the-fly from Python to R only upon request.: [evidence span withheld — anchor status: not_found]
- [readme] Spectra backend list: [evidence span withheld — anchor status: external_doc]
- [readme] install/config: "- [*matchms*](https://github.com/matchms) 0.31
- [*spectrum_utils*](https://github.com/bittremieux-lab/spectrum_utils) 0.3.2
- *numpy* 2.2.0"
- [readme] Python version: "*SpectriPy* needs Python (version >= 3.12) to be installed on the system. All
necessary Python libraries (listed below) are automatically installed by"
