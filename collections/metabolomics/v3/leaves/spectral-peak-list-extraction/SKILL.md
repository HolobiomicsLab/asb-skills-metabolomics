---
name: spectral-peak-list-extraction
description: Use when peak tables (m/z + intensity pairs) must be pulled out of spectra
  objects across the R/Python boundary — e.g. an R Spectra object pointing at a Python-side
  matchms object — and only requested subsets should be translated.
when_to_use_negative:
- When the peak data already resides fully in R (e.g. a MsBackendDataFrame-backed
  Spectra) and no cross-language translation is needed
- When a full data copy is actually required (e.g. the Python session will be closed
  before further use) — on-the-fly translation keeps data on the Python side
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: SpectriPy
  role: Provides the MsBackendPy backend class that lets an R Spectra object access
    MS data held as a Python object, translating requested subsets on-the-fly
  repo: https://github.com/rformassspectrometry/SpectriPy
- name: Spectra
  role: R package defining the Spectra object and backend infrastructure (including
    on-the-fly backends) through which peaksData access is performed
  repo: https://github.com/RforMassSpectrometry/Spectra
- name: matchms
  role: Python library holding the source spectrum objects (e.g. matchms.Spectrum)
    from which m/z and intensity peak values are extracted
  repo: https://github.com/matchms
- name: reticulate
  role: Manages the Python session and py_require()-managed library installation that
    bridges the R and Python data objects
  repo: https://rstudio.github.io/reticulate
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:c2e507845bbc8d4d69635a109dc67a8b13d0e1e773fba2e828b06aea5513c4a1
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spectral-peak-list-extraction

## Summary

Extract two-column (m/z, intensity) peak tables from MS spectra objects, including lazy on-the-fly extraction from a Python matchms.Spectrum held in a reticulate-managed Python session via SpectriPy's MsBackendPy backend, without copying the underlying peak data.

## When to use

Use when peak tables (m/z + intensity pairs) must be pulled out of spectra objects across the R/Python boundary — e.g. an R Spectra object pointing at a Python-side matchms object — and only requested subsets should be translated.

## When NOT to use

- When the peak data already resides fully in R (e.g. a MsBackendDataFrame-backed Spectra) and no cross-language translation is needed
- When a full data copy is actually required (e.g. the Python session will be closed before further use) — on-the-fly translation keeps data on the Python side

## Inputs

- Python-side matchms.Spectrum object (e.g. 'mgf_py') created in a reticulate-managed Python session
- R Spectra object instantiated with source = MsBackendPy() referencing the Python object by name
- Python environment requirements: matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0 (Python >= 3.12)

## Outputs

- Two-column (m/z, intensity) numeric peak table for the requested spectrum subset, e.g. exported to peaks_data_spectrum1.csv
- Optionally, a translated R-side representation of the subset without copying the full Python data object

## How to apply

Configure the Python environment through reticulate's py_require() so the tested library versions (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0) are available, and create or load the Python-side data object (e.g. a matchms.Spectrum such as 'mgf_py'). In R, attach Spectra and SpectriPy and instantiate Spectra('mgf_py', source = MsBackendPy()) so the backend points at the Python object without copying peak data. Access peaksData on a subset (e.g. s[1]) to trigger on-the-fly translation of only the requested spectrum's m/z and intensity values from Python to R, then export the two-column table (e.g. to peaks_data_spectrum1.csv). The rationale is lazy, subset-level translation: all data from the referenced Python object is accessible in R, but the entire data or subsets are translated on-the-fly only upon request, avoiding full duplication of peak data in both sessions.

## Related tools

- **SpectriPy** (Provides the MsBackendPy backend class that lets an R Spectra object access MS data held as a Python object, translating requested subsets on-the-fly) — https://github.com/rformassspectrometry/SpectriPy
- **Spectra** (R package defining the Spectra object and backend infrastructure (including on-the-fly backends) through which peaksData access is performed) — https://github.com/RforMassSpectrometry/Spectra
- **matchms** (Python library holding the source spectrum objects (e.g. matchms.Spectrum) from which m/z and intensity peak values are extracted) — https://github.com/matchms
- **reticulate** (Manages the Python session and py_require()-managed library installation that bridges the R and Python data objects) — https://rstudio.github.io/reticulate

## Examples

```
s <- Spectra("mgf_py", source = MsBackendPy()); pd <- peaksData(s[1]); write.csv(pd[[1]], "peaks_data_spectrum1.csv", row.names = FALSE)
```

## Evaluation signals

- The exported CSV contains exactly the peak count held by the referenced Python object (e.g. 'mgf_py') for the subset spectrum
- Each row's m/z and intensity values match those stored in the Python session object value-for-value
- Only the requested subset (e.g. spectrum [1]) was translated — the CSV contains one spectrum's peaks, not the full collection
- The R Spectra object was created without copying peak data (backend remains a reference to the Python-side object)

## Limitations

- The first Python-backed call can be slower because the Python setup has to be initialized (reticulate environment startup)
- Requires Python >= 3.12 and the exact tested library versions (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0); if RETICULATE_PYTHON or RETICULATE_PYTHON_ENV variables are defined, all required libraries must be installed manually since reticulate will not install them automatically
- Data stays on the Python side — subsets are translated only upon request, so the R object is not a standalone copy

## Evidence

- [abstract] provides functionality to efficiently translate between R and Python MS data structures: "provides functionality to efficiently translate between R and Python MS data structures"
- [readme] By wrapping Python functionality into R functions, SpectriPy allows a seamless integration of Python libraries into R: "By wrapping Python functionality into R functions, *SpectriPy* allows a
seamless integration of Python libraries into R."
- [readme] Spectra provides different implementations (backends) to store mass spectrometry data: "*Spectra* provides different implementations (backends) to store mass
spectrometry data."
- [readme] The SpectriPy package provides the compareSpectriPy() function; the first call takes longer because the Python setup has to be initialized: "Note also that the first `compareSpectriPy()` call takes usually a little longer
because the Python setup has to be initialized."
