---
name: python-to-r-spectra-conversion
description: Use when a workflow has produced matchms.Spectrum objects in Python (e.g.,
  after an rspec_to_pyspec() step) and the downstream analysis or round-trip validation
  must continue in R with a Spectra::Spectra object.
when_to_use_negative:
- The data are already R Spectra::Spectra objects and no Python-side manipulation
  has occurred — use Spectra's native R functionality instead
- The Python objects are spectrum_utils.spectrum.MsmsSpectrum objects intended for
  plotting rather than conversion back into the R workflow
- The goal is similarity scoring rather than object conversion — compareSpectriPy()
  handles the R-to-Python translation internally
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: SpectriPy
  role: Provides pyspec_to_rspec() and defaultSpectraVariableMapping() to convert
    matchms.Spectrum objects back to Spectra::Spectra with metadata renaming
  repo: https://github.com/rformassspectrometry/SpectriPy
- name: reticulate
  role: R package used to load and access the Python matchms.Spectrum objects inside
    the R session
  repo: https://rstudio.github.io/reticulate
- name: Spectra
  role: Target R object model (Spectra::Spectra) whose peaks data and spectra variables
    are rebuilt and validated
  repo: https://github.com/RforMassSpectrometry/Spectra
- name: matchms
  role: Python source library whose Spectrum objects are the conversion input (pythonLibrary='matchms')
  repo: https://github.com/matchms
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_001
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:18b89acfa0c1efcd4c7e2b5d874cd3e51e5e16c9fa22964450839292ce18257a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rformassspectrometry/SpectriPy
  tools:
  - Python
  - R
  - Spectra
  - SpectriPy
  - matchms
  - reticulate
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# python-to-r-spectra-conversion

## Summary

Rebuilds R Spectra::Spectra objects from Python matchms.Spectrum objects using SpectriPy's pyspec_to_rspec(), including renaming and reformatting of spectra metadata (MS level, precursorMz, retention times) via defaultSpectraVariableMapping() across the reticulate bridge.

## When to use

Use when a workflow has produced matchms.Spectrum objects in Python (e.g., after an rspec_to_pyspec() step) and the downstream analysis or round-trip validation must continue in R with a Spectra::Spectra object.

## When NOT to use

- The data are already R Spectra::Spectra objects and no Python-side manipulation has occurred — use Spectra's native R functionality instead
- The Python objects are spectrum_utils.spectrum.MsmsSpectrum objects intended for plotting rather than conversion back into the R workflow
- The goal is similarity scoring rather than object conversion — compareSpectriPy() handles the R-to-Python translation internally

## Inputs

- Python matchms.Spectrum objects exposed to R via the reticulate package
- Original R Spectra::Spectra object used as round-trip reference (peaks matrices and spectra variables)

## Outputs

- Reconstructed R Spectra::Spectra object
- Metadata comparison table listing each spectra variable with original value, round-tripped value, and match flag
- Persisted RDS file (e.g., spectra_roundtrip.rds)

## How to apply

Load the Python matchms.Spectrum objects into the R session via the reticulate package, then call pyspec_to_rspec() with defaultSpectraVariableMapping() and pythonLibrary='matchms' so that Python metadata fields are renamed and reformatted to their Spectra equivalents (MS level, precursorMz, retention times, compound name). Validate the reconstruction by comparing the peaks data (m/z and intensity matrices) against the original R Spectra object with all.equal at standard floating-point tolerance, and compare mapped spectra variables element-wise, assembling a per-variable comparison table with original value, round-tripped value, and a match flag. Accept the round trip only if all m/z/intensity pairs match within numeric tolerance and every mapped metadata field is identical. Save the reconstructed Spectra object and comparison table (e.g., spectra_roundtrip.rds) with saveRDS.

## Related tools

- **SpectriPy** (Provides pyspec_to_rspec() and defaultSpectraVariableMapping() to convert matchms.Spectrum objects back to Spectra::Spectra with metadata renaming) — https://github.com/rformassspectrometry/SpectriPy
- **reticulate** (R package used to load and access the Python matchms.Spectrum objects inside the R session) — https://rstudio.github.io/reticulate
- **Spectra** (Target R object model (Spectra::Spectra) whose peaks data and spectra variables are rebuilt and validated) — https://github.com/RforMassSpectrometry/Spectra
- **matchms** (Python source library whose Spectrum objects are the conversion input (pythonLibrary='matchms')) — https://github.com/matchms

## Examples

```
spectra_r <- pyspec_to_rspec(py$spec_list, mapping = defaultSpectraVariableMapping(), pythonLibrary = "matchms"); all.equal(peaksData(spectra_r), peaksData(orig))
```

## Evaluation signals

- all.equal on the m/z and intensity matrices of the round-tripped Spectra object vs. the original, within standard floating-point tolerance
- Every mapped spectra variable (MS level, precursorMz, retention times, compound name) is identical element-wise between original and reconstructed objects
- Metadata comparison table has a TRUE match flag for every spectra variable row
- The reconstructed object is a valid Spectra::Spectra instance that can be persisted with saveRDS and reloaded

## Limitations

- Metadata fidelity depends on defaultSpectraVariableMapping(); fields not covered by the mapping may be dropped or renamed incorrectly
- First reticulate/Python initialization (SpectriPy requires Python >= 3.12, matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0) adds overhead and can fail if environment variables like RETICULATE_PYTHON point to an environment missing required libraries
- Floating-point round-tripping means peak values must be compared with numeric tolerance, not strict equality

## Evidence

- [methods] Conversion between R Spectra::Spectra and Python matchms/spectrum_utils spectrum objects: "contains
functions to convert between R's `Spectra::Spectra` objects and
`matchms.Spectrum` and `spectrum_utils.spectrum.MsmsSpectrum` objects"
- [abstract] Bidirectional translation including metadata renaming: [evidence span withheld — anchor status: not_found]
- [abstract] Efficient R/Python data structure translation: "provides functionality to efficiently translate between R and Python MS data structures"
- [readme] reticulate as the R/Python execution bridge: "create custom functions or workflows on `Spectra` objects in Python
and executing them in R using the *reticulate* R package, and vice versa"
