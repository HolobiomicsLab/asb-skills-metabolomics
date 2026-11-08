---
name: cross-language-spectrum-conversion
description: Use when an R-based Spectra workflow must consume or hand off MS2 peak
  tables to/from Python matchms or spectrum_utils objects in one session, without
  duplicating data across languages.
when_to_use_negative:
- The analysis stays entirely in one language — use Spectra::compareSpectra() in R
  or matchms natively in Python instead of converting objects
- Input is already a raw data file (mzML/MGF) needing parsing rather than object-level
  conversion between sessions
- The task is similarity scoring rather than data access/conversion — use compareSpectriPy()
  with parameters like CosineGreedyParam or ModifiedCosine instead
edam_operation: http://edamontology.org/operation_2422
tools:
- name: SpectriPy
  role: Provides MsBackendPy backend and conversion functions between R Spectra::Spectra
    and Python spectrum objects; manages the Python environment via reticulate py_require()
  repo: https://github.com/rformassspectrometry/SpectriPy
- name: Spectra
  role: Defines the R Spectra object class and backend interface that MsBackendPy
    plugs into for lazy on-the-fly data access
  repo: https://github.com/RforMassSpectrometry/Spectra
- name: matchms
  role: Python library whose matchms.Spectrum objects hold the MS2 peak data accessed
    or converted from R (tested version 0.31)
  repo: https://github.com/matchms
- name: spectrum_utils
  role: Python library whose MsmsSpectrum objects are an alternative target/source
    for R-Python translation (tested version 0.3.2)
  repo: https://github.com/bittremieux-lab/spectrum_utils
- name: reticulate
  role: R package that manages Python library installation (py_require()) and executes
    Python code from the R session
  repo: https://rstudio.github.io/reticulate
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:8c7d2aaba385c1fc5359ce0127ee9d60426167ce9bcbe9fe4b38f175d3cb8331
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
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cross-language-spectrum-conversion

## Summary

Translates MS spectrum data between R's Spectra::Spectra objects and Python's matchms.Spectrum / spectrum_utils.spectrum.MsmsSpectrum objects using SpectriPy, including lazy, on-the-fly access to Python-held peak data in R via the MsBackendPy backend.

## When to use

Use when an R-based Spectra workflow must consume or hand off MS2 peak tables to/from Python matchms or spectrum_utils objects in one session, without duplicating data across languages.

## When NOT to use

- The analysis stays entirely in one language — use Spectra::compareSpectra() in R or matchms natively in Python instead of converting objects
- Input is already a raw data file (mzML/MGF) needing parsing rather than object-level conversion between sessions
- The task is similarity scoring rather than data access/conversion — use compareSpectriPy() with parameters like CosineGreedyParam or ModifiedCosine instead

## Inputs

- R Spectra::Spectra object (e.g. built from a DataFrame with msLevel, precursorMz, mz and intensity peak lists)
- Python matchms.Spectrum object (e.g. created in the reticulate-managed Python session, e.g. 'mgf_py')
- Python spectrum_utils.spectrum.MsmsSpectrum object
- MGF library spectra (e.g. for LC-MS/MS annotation workflows)

## Outputs

- R Spectra::Spectra object translated from a Python spectrum object
- Python matchms.Spectrum / MsmsSpectrum object translated from an R Spectra object
- On-demand two-column (m/z, intensity) peak tables in R translated from the Python object
- CSV exports of translated peak subsets (e.g. peaks_data_spectrum1.csv)

## How to apply

Configure the Python environment through reticulate's py_require() so the tested library stack (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0, Python >= 3.12) is available, then attach SpectriPy and Spectra in R. For full conversion, use SpectriPy's functions to translate between R Spectra::Spectra and matchms.Spectrum / MsmsSpectrum objects; for zero-copy access, instantiate Spectra('mgf_py', source = MsBackendPy()) where the backend points at a Python-side object by name and only translates requested subsets (e.g. peaksData of spectrum [1]) from Python to R on the fly. Wrap custom Python functionality into R functions or execute Python code on Spectra objects via reticulate, and vice versa. The first call initializes the Python setup, so expect initial latency; subsequent calls are faster.

## Related tools

- **SpectriPy** (Provides MsBackendPy backend and conversion functions between R Spectra::Spectra and Python spectrum objects; manages the Python environment via reticulate py_require()) — https://github.com/rformassspectrometry/SpectriPy
- **Spectra** (Defines the R Spectra object class and backend interface that MsBackendPy plugs into for lazy on-the-fly data access) — https://github.com/RforMassSpectrometry/Spectra
- **matchms** (Python library whose matchms.Spectrum objects hold the MS2 peak data accessed or converted from R (tested version 0.31)) — https://github.com/matchms
- **spectrum_utils** (Python library whose MsmsSpectrum objects are an alternative target/source for R-Python translation (tested version 0.3.2)) — https://github.com/bittremieux-lab/spectrum_utils
- **reticulate** (R package that manages Python library installation (py_require()) and executes Python code from the R session) — https://rstudio.github.io/reticulate

## Examples

```
library(Spectra); library(SpectriPy); sps <- Spectra("mgf_py", source = MsBackendPy()); peaks_data_spectrum1 <- peaksData(sps[1])
```

## Evaluation signals

- The exported peak table (e.g. peaks_data_spectrum1.csv) contains exactly the peak count held by the referenced Python object
- m/z and intensity values in the R-side output match the Python session object's values value-for-value
- Only the requested spectrum subset was translated — no full-copy of the Python data appears in R memory
- First SpectriPy call initializes Python setup (slower); subsequent calls return without re-initialization
- compareSpectriPy()-style round-trips return a numeric matrix of the expected dimensions (e.g. 4x2 for four query spectra vs two Caffeine spectra)

## Limitations

- Requires Python >= 3.12 and reticulate >= 1.41 (py_require()); if RETICULATE_PYTHON or RETICULATE_PYTHON_ENV is set, all listed libraries (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0) must be installed manually
- The first call in a session is slower because the Python setup has to be initialized
- On-the-fly translation means each access re-triggers Python-to-R translation of the requested subset

## Evidence

- [abstract] SpectriPy can convert between R's Spectra::Spectra and Python's matchms.Spectrum and spectrum_utils.spectrum.MsmsSpectrum objects: [evidence span withheld — anchor status: not_found]
- [abstract] SpectriPy allows directly applying spectral similarity, filtering, and normalization routines from the Python matchms library on MS data in R: "includes functionality to directly apply spectral similarity, filtering, normalization, etc. routines from the Python matchms library on MS data in R"
- [readme] R and Python spectral objects are easily translated and available in one workflow via reticulate: "R and Python spectral objects are easily translated and available
in one workflow (i.e., a quarto document)"
- [other] All data from the referenced Python data object is accessible in R, with data translated on-the-fly only upon request: [evidence span withheld — anchor status: not_found]
- [readme] Tested Python dependency versions for the reticulate-managed environment: "- [*matchms*](https://github.com/matchms) 0.31
- [*spectrum_utils*](https://github.com/bittremieux-lab/spectrum_utils) 0.3.2
- *numpy* 2.2.0"
