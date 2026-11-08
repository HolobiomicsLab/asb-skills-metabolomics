---
name: reticulate-cross-language-object-mapping
description: Use when an R-based Spectra workflow must hand MS2 peak data (mz/intensity
  matrices plus spectra variables like msLevel, precursorMz, rtime) to Python matchms
  functions, or matchms results must be returned to R, without intermediate file serialization.
when_to_use_negative:
- The analysis can stay entirely in one language — SpectriPy conversion is only needed
  when mixing R Spectra and Python matchms/spectrum_utils functionality in one workflow
- Data are raw instrument files (e.g., mzML) rather than in-memory spectrum objects
  — use Spectra backends or matchms file readers first, not the object-conversion
  functions
- The Python environment is a custom virtualenv/conda but does not have ALL required
  libraries (matchms, spectrum_utils, numpy) installed — reticulate will not auto-install
  them when RETICULATE_PYTHON/RETICULATE_PYTHON_ENV is set
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SpectriPy
  role: Provides rspec_to_pyspec()/pyspec_to_rspec() conversion functions and defaultSpectraVariableMapping()
    that translate between R Spectra::Spectra and Python matchms.Spectrum objects,
    renaming/reformatting spectra metadata
  repo: github.com/rformassspectrometry/SpectriPy
- name: reticulate
  role: R package used to load Python matchms.Spectrum objects into the R session
    and to execute custom Python functions on Spectra objects in R (and vice versa);
    manages the Python environment via py_require()
  repo: https://rstudio.github.io/reticulate
- name: Spectra
  role: Defines the R-side Spectra::Spectra object and spectra variables (MS level,
    precursorMz, retention times) that serve as source/target of the conversion
  repo: github.com/RforMassSpectrometry/Spectra
- name: matchms
  role: Python library whose Spectrum objects are the conversion target/source; required
    version 0.31
  repo: https://github.com/matchms
- name: spectrum_utils
  role: Alternative Python conversion target via spectrum_utils.spectrum.MsmsSpectrum
    objects; required version 0.3.2
  repo: https://github.com/bittremieux-lab/spectrum_utils
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:f5a920a08028af33db158d8c9fd48d3a08a4d5ffe0301c4f3e621c5f13a80f7c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Spectra
  - SpectriPy
  - matchms
  - reticulate
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# reticulate-cross-language-object-mapping

## Summary

Translate mass spectrometry data objects between R Spectra::Spectra and Python matchms.Spectrum (or spectrum_utils.MsmsSpectrum) objects within a single workflow using the SpectriPy package and the reticulate R package, including renaming/reformatting of spectra metadata such as MS level and retention times.

## When to use

Use when an R-based Spectra workflow must hand MS2 peak data (mz/intensity matrices plus spectra variables like msLevel, precursorMz, rtime) to Python matchms functions, or matchms results must be returned to R, without intermediate file serialization.

## When NOT to use

- The analysis can stay entirely in one language — SpectriPy conversion is only needed when mixing R Spectra and Python matchms/spectrum_utils functionality in one workflow
- Data are raw instrument files (e.g., mzML) rather than in-memory spectrum objects — use Spectra backends or matchms file readers first, not the object-conversion functions
- The Python environment is a custom virtualenv/conda but does not have ALL required libraries (matchms, spectrum_utils, numpy) installed — reticulate will not auto-install them when RETICULATE_PYTHON/RETICULATE_PYTHON_ENV is set

## Inputs

- R Spectra::Spectra object (MsBackendDataFrame or other backend) with mz/intensity peak lists and spectra variables (msLevel, precursorMz, rtime, compound name)
- Python matchms.Spectrum or spectrum_utils.spectrum.MsmsSpectrum objects present in the reticulate-connected Python session

## Outputs

- Python matchms.Spectrum objects converted from R Spectra::Spectra (via rspec_to_pyspec)
- Reconstructed R Spectra::Spectra object (via pyspec_to_rspec)
- Metadata comparison table mapping each spectra variable to its original and round-tripped value with a match flag (e.g., saved as spectra_roundtrip.rds)

## How to apply

Install SpectriPy via BiocManager (requires Python >= 3.12; reticulate >= 1.41 auto-installs matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0 via py_require(), or set RETICULATE_PYTHON/RETICULATE_PYTHON_ENV for a preconfigured environment). In R, load Spectra and SpectriPy, then call rspec_to_pyspec() to convert a Spectra::Spectra object into matchms.Spectrum objects and pyspec_to_rspec() with defaultSpectraVariableMapping() and pythonLibrary='matchms' to rebuild a Spectra object from Python spectrum objects retrieved through reticulate. The conversion functions handle required renaming and reformatting of spectra metadata (e.g., MS level and retention times) according to the variable mapping. Optionally, custom Python functions can be written to operate on Spectra objects and executed in R via reticulate, and vice versa. Validate a round trip by comparing mz/intensity peak matrices with all.equal under numeric tolerance and checking element-wise identity of mapped spectra variables (MS level, precursorMz, retention times, compound name) between original and reconstructed objects.

## Related tools

- **SpectriPy** (Provides rspec_to_pyspec()/pyspec_to_rspec() conversion functions and defaultSpectraVariableMapping() that translate between R Spectra::Spectra and Python matchms.Spectrum objects, renaming/reformatting spectra metadata) — github.com/rformassspectrometry/SpectriPy
- **reticulate** (R package used to load Python matchms.Spectrum objects into the R session and to execute custom Python functions on Spectra objects in R (and vice versa); manages the Python environment via py_require()) — https://rstudio.github.io/reticulate
- **Spectra** (Defines the R-side Spectra::Spectra object and spectra variables (MS level, precursorMz, retention times) that serve as source/target of the conversion) — github.com/RforMassSpectrometry/Spectra
- **matchms** (Python library whose Spectrum objects are the conversion target/source; required version 0.31) — https://github.com/matchms
- **spectrum_utils** (Alternative Python conversion target via spectrum_utils.spectrum.MsmsSpectrum objects; required version 0.3.2) — https://github.com/bittremieux-lab/spectrum_utils

## Examples

```
library(Spectra); library(SpectriPy); py <- rspec_to_pyspec(s); s2 <- pyspec_to_rspec(py, pythonLibrary = "matchms"); all.equal(peaksData(s), peaksData(s2))
```

## Evaluation signals

- Round-tripped Spectra object's mz and intensity peak matrices match the original via all.equal() within standard floating-point tolerance
- Every mapped spectra variable (MS level, precursorMz, retention times, compound name) is element-wise identical between original and reconstructed Spectra objects
- Metadata comparison table shows a match flag for all listed spectra variables
- First Python-dependent call succeeds only after reticulate has initialized the Python setup (expect slightly longer runtime on first call)
- Python dependency versions match the tested set: matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0

## Limitations

- SpectriPy requires Python >= 3.12 and reticulate >= 1.41 (py_require()-based library management); older reticulate versions are not supported
- If RETICULATE_PYTHON or RETICULATE_PYTHON_ENV is set, all required Python libraries (matchms, spectrum_utils, numpy) must be installed manually — reticulate will not install them automatically
- Metadata fidelity depends on the spectra variable mapping: fields must be renamed/reformatted (e.g., MS level, retention times) during conversion, and unmapped variables may not survive the round trip
- Floating-point representation across R/Python means peak values must be compared with numeric tolerance rather than exact equality

## Evidence

- [other] SpectriPy's rspec_to_pyspec() and pyspec_to_rspec() conversion functions also handle the conversion, and any required renaming and reformatting of spectra metadata: [evidence span withheld — anchor status: not_found]
- [abstract] SpectriPy converts between R and Python MS objects: [evidence span withheld — anchor status: not_found]
- [readme] SpectriPy needs Python >= 3.12; reticulate auto-installs dependencies: "*SpectriPy* needs Python (version >= 3.12) to be installed on the system. All
necessary Python libraries (listed below) are automatically installed by the"
- [readme] Required Python library versions: "The required Python libraries with the suggested and tested
versions are:

- [*matchms*](https://github.com/matchms) 0.31
- [*spectrum_utils*](https://github.com/bittremieux-lab/spectrum_utils)"
- [methods] Custom Python workflows on Spectra objects via reticulate: "create custom functions or workflows on `Spectra` objects in Python
and executing them in R using the *reticulate* R package, and vice versa"
