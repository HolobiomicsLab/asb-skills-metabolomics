---
name: rs-python-spectra-object-conversion
description: Use when MS2 peak data and metadata (msLevel, precursorMz, retention
  times, compound name/id) live in an R Spectra::Spectra object but the next processing
  step is a Python matchms or spectrum_utils routine.
when_to_use_negative:
- Data already exists as Python matchms.Spectrum objects and only needs to move back
  to R — use pyspec_to_rspec() instead
- The task is similarity scoring or filtering rather than data exchange — compareSpectriPy()
  and filterSpectriPy() perform the conversion internally, so a manual conversion
  step is redundant
- No Python environment is available or reticulate < 1.41 is installed, since SpectriPy's
  Python library management relies on py_require() from reticulate 1.41
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SpectriPy
  role: Provides rspec_to_pyspec() and the spectraVariableMapping definitions used
    to translate R Spectra::Spectra objects into Python spectrum objects
  repo: https://github.com/RforMassSpectrometry/SpectriPy
- name: Spectra
  role: Defines the R-side Spectra::Spectra object holding the MS2 peaks and spectra
    variables to be converted
  repo: https://github.com/RforMassSpectrometry/Spectra
- name: matchms
  role: Target Python library whose matchms.Spectrum objects are produced by the conversion;
    tested version 0.31
  repo: https://github.com/matchms
- name: spectrum_utils
  role: Alternative conversion target producing spectrum_utils.spectrum.MsmsSpectrum
    objects; tested version 0.3.2
  repo: https://github.com/bittremieux-lab/spectrum_utils
- name: reticulate
  role: R package that initializes the Python environment and mediates the cross-language
    object transfer
  repo: https://rstudio.github.io/reticulate
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:68691e80b3f35a728a5fbf57f4b9287568b787317c061a3d42d2a1ec39db30bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RforMassSpectrometry/SpectriPy
  tools:
  - Python
  - R
  - Spectra
  - SpectriPy
  - matchms
  - reticulate
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# rs-python-spectra-object-conversion

## Summary

Converts R Spectra::Spectra objects into Python matchms.Spectrum (or spectrum_utils MsmsSpectrum) objects via SpectriPy's rspec_to_pyspec(), translating peak arrays and metadata across the reticulate bridge.

## When to use

Use when MS2 peak data and metadata (msLevel, precursorMz, retention times, compound name/id) live in an R Spectra::Spectra object but the next processing step is a Python matchms or spectrum_utils routine.

## When NOT to use

- Data already exists as Python matchms.Spectrum objects and only needs to move back to R — use pyspec_to_rspec() instead
- The task is similarity scoring or filtering rather than data exchange — compareSpectriPy() and filterSpectriPy() perform the conversion internally, so a manual conversion step is redundant
- No Python environment is available or reticulate < 1.41 is installed, since SpectriPy's Python library management relies on py_require() from reticulate 1.41

## Inputs

- R Spectra::Spectra object with MS2 peak m/z and intensity lists and spectra metadata (msLevel, precursorMz, rtime, name, id)
- spectraVariableMapping / defaultSpectraVariableMapping metadata mapping definition
- Python environment configured with matchms 0.31, spectrum_utils 0.3.2, and numpy 2.2.0 via reticulate

## Outputs

- List of Python matchms.Spectrum objects (or spectrum_utils.spectrum.MsmsSpectrum objects)
- Tabular CSV of applied metadata variable mapping (R column name vs. matchms metadata key)
- Optionally a serialized pickle file of the converted spectra

## How to apply

Initialize the Python environment through the reticulate R package (SpectriPy requires Python >= 3.12 and manages libraries via reticulate's py_require(), or set RETICULATE_PYTHON/RETICULATE_PYTHON_ENV when using a system Python, virtualenv, or conda environment). Load the Spectra object (e.g. caffeine and 1-methylhistidine MS2 spectra with mz/intensity peak lists and metadata such as msLevel, precursorMz, name, id) and call rspec_to_pyspec() with the pythonLibrary argument set to 'matchms' (or 'spectrum_utils' for MsmsSpectrum objects). SpectriPy renames and reformats spectra metadata columns—MS level, precursor m/z, retention time, compound name/ID—according to the defaultSpectraVariableMapping / spectraVariableMapping definitions so keys comply with matchms metadata conventions. The result is a list of matchms.Spectrum objects whose peak m/z and intensity arrays reproduce the R source; the applied variable mapping (R column name vs. matchms metadata key) can be written to a tabular CSV and the objects serialized (e.g. pickle) for downstream Python use.

## Related tools

- **SpectriPy** (Provides rspec_to_pyspec() and the spectraVariableMapping definitions used to translate R Spectra::Spectra objects into Python spectrum objects) — https://github.com/RforMassSpectrometry/SpectriPy
- **Spectra** (Defines the R-side Spectra::Spectra object holding the MS2 peaks and spectra variables to be converted) — https://github.com/RforMassSpectrometry/Spectra
- **matchms** (Target Python library whose matchms.Spectrum objects are produced by the conversion; tested version 0.31) — https://github.com/matchms
- **spectrum_utils** (Alternative conversion target producing spectrum_utils.spectrum.MsmsSpectrum objects; tested version 0.3.2) — https://github.com/bittremieux-lab/spectrum_utils
- **reticulate** (R package that initializes the Python environment and mediates the cross-language object transfer) — https://rstudio.github.io/reticulate

## Examples

```
library(Spectra); library(SpectriPy); py_spectra <- rspec_to_pyspec(caf, pythonLibrary = "matchms")
```

## Evaluation signals

- Reload the serialized pickle in Python and confirm each matchms.Spectrum reproduces the source spectra's peak m/z and intensity arrays
- Verify every mapped metadata field (MS level, precursor m/z, retention time, compound name/id) is present under its renamed matchms metadata key
- Confirm the exported variable-mapping CSV covers all renamed spectra variables and matches the defaultSpectraVariableMapping entries
- Check that metadata values survive round-tripping (e.g. precursorMz 195.0877 for caffeine, 170.0924 for 1-methylhistidine, HMDB ids preserved)

## Limitations

- Requires Python >= 3.12; if RETICULATE_PYTHON or RETICULATE_PYTHON_ENV is defined, all required Python libraries (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0) must be installed manually since reticulate will not install them automatically
- The first call takes longer because the Python setup has to be initialized
- Metadata must be renamed/reformatted per spectraVariableMapping; arbitrary R spectra variables not covered by the mapping may not transfer under expected matchms keys

## Evidence

- [abstract] It can convert between R's Spectra::Spectra and Python's matchms.Spectrum and spectrum_utils.spectrum.MsmsSpectrum objects: [evidence span withheld — anchor status: not_found]
- [abstract] provides functionality to efficiently translate between R and Python MS data structures: "provides functionality to efficiently translate between R and Python MS data structures"
- [methods] contains functions to convert between R's Spectra::Spectra objects and matchms.Spectrum and spectrum_utils.spectrum.MsmsSpectrum objects: "contains
functions to convert between R's `Spectra::Spectra` objects and
`matchms.Spectrum` and `spectrum_utils.spectrum.MsmsSpectrum` objects"
- [readme] SpectriPy needs Python (version >= 3.12) to be installed on the system: "*SpectriPy* needs Python (version >= 3.12) to be installed on the system."
- [readme] R and Python spectral objects are easily translated and available in one workflow: "R and Python spectral objects are easily translated and available
in one workflow (i.e., a quarto document)"
