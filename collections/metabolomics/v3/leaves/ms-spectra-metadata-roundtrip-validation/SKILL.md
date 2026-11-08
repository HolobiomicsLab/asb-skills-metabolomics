---
name: ms-spectra-metadata-roundtrip-validation
description: Use when Python matchms.Spectrum objects produced by an upstream rspec_to_pyspec
  conversion must be rebuilt as R Spectra::Spectra objects, and you must prove no
  loss in peaks data or metadata.
when_to_use_negative:
- The goal is similarity scoring or filtering rather than conversion — use compareSpectriPy()
  with e.g. CosineGreedyParam or filterSpectriPy instead
- The data already lives entirely in one language (pure R Spectra or pure Python matchms
  pipeline) with no cross-language boundary to validate
- Conversion targets spectrum_utils.spectrum.MsmsSpectrum objects rather than matchms.Spectrum
  — a different SpectriPy conversion path applies
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SpectriPy
  role: Provides pyspec_to_rspec(), rspec_to_pyspec(), and defaultSpectraVariableMapping()
    for the R↔Python conversion being validated
  repo: https://github.com/rformassspectrometry/SpectriPy
- name: Spectra
  role: Defines the R Spectra::Spectra target object and the spectra variables (MS
    level, precursorMz, retention times) compared after round trip
  repo: https://github.com/RforMassSpectrometry/Spectra
- name: matchms
  role: Python library whose matchms.Spectrum objects are converted back to R; selected
    via pythonLibrary='matchms'
  repo: https://github.com/matchms
- name: reticulate
  role: Bridges the R session to Python so matchms.Spectrum objects can be loaded
    and passed to pyspec_to_rspec()
  repo: https://rstudio.github.io/reticulate
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:adb1cceaf4a467e7dfe6a967bfa4e65bc4ae8e3a6059837aa3f883e9f13398b2
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# MS spectra metadata roundtrip validation

## Summary

Validates the fidelity of converting matchms.Spectrum objects back into R Spectra::Spectra objects with SpectriPy's pyspec_to_rspec(), by comparing peaks data (mz/intensity matrices) and mapped spectra variables (MS level, precursorMz, retention times) against the original R object after an rspec_to_pyspec → pyspec_to_rspec round trip.

## When to use

Use when Python matchms.Spectrum objects produced by an upstream rspec_to_pyspec conversion must be rebuilt as R Spectra::Spectra objects, and you must prove no loss in peaks data or metadata.

## When NOT to use

- The goal is similarity scoring or filtering rather than conversion — use compareSpectriPy() with e.g. CosineGreedyParam or filterSpectriPy instead
- The data already lives entirely in one language (pure R Spectra or pure Python matchms pipeline) with no cross-language boundary to validate
- Conversion targets spectrum_utils.spectrum.MsmsSpectrum objects rather than matchms.Spectrum — a different SpectriPy conversion path applies

## Inputs

- Python matchms.Spectrum objects (e.g. reticulate handle to list of spectra produced by rspec_to_pyspec)
- Original R Spectra::Spectra object used as the round-trip reference
- Spectra variable mapping via defaultSpectraVariableMapping()

## Outputs

- Reconstructed R Spectra::Spectra object (saved to spectra_roundtrip.rds via saveRDS)
- Metadata comparison table with one row per spectra variable, original and round-tripped values, and a match flag
- all.equal() result for mz/intensity peaks matrices

## How to apply

Load the matchms.Spectrum objects from the upstream rspec_to_pyspec step into R via reticulate, then call pyspec_to_rspec() with defaultSpectraVariableMapping() and pythonLibrary='matchms' to rebuild a Spectra object. Compare the peaks data (mz and intensity matrices) of the round-tripped object against the original with all.equal() using standard floating-point tolerance. Compare mapped spectra variables (MS level, precursorMz, retention times, compound name) element-wise between original and reconstructed objects, recording mismatches. Assemble a metadata comparison table listing each spectra variable with its original value, round-trip value, and a match flag. Rationale: SpectriPy's conversion functions handle renaming and reformatting of spectra metadata across languages (e.g. MS level, retention times), so only a full field-by-field round-trip comparison can confirm nothing was dropped or mistranslated. Save the reconstructed Spectra object and comparison table to spectra_roundtrip.rds with saveRDS.

## Related tools

- **SpectriPy** (Provides pyspec_to_rspec(), rspec_to_pyspec(), and defaultSpectraVariableMapping() for the R↔Python conversion being validated) — https://github.com/rformassspectrometry/SpectriPy
- **Spectra** (Defines the R Spectra::Spectra target object and the spectra variables (MS level, precursorMz, retention times) compared after round trip) — https://github.com/RforMassSpectrometry/Spectra
- **matchms** (Python library whose matchms.Spectrum objects are converted back to R; selected via pythonLibrary='matchms') — https://github.com/matchms
- **reticulate** (Bridges the R session to Python so matchms.Spectrum objects can be loaded and passed to pyspec_to_rspec()) — https://rstudio.github.io/reticulate

## Examples

```
s_rt <- pyspec_to_rspec(s_py, mapping = defaultSpectraVariableMapping(), pythonLibrary = "matchms"); all.equal(peaksData(s_rt), peaksData(s_orig))
```

## Evaluation signals

- all.equal() on the mz and intensity matrices of original vs. round-tripped Spectra object passes within standard floating-point tolerance
- Every mapped spectra variable (MS level, precursorMz, retention times, compound name) is identical element-wise between original and reconstructed objects
- All match flags in the metadata comparison table are TRUE before accepting the round trip
- Number of spectra and peak counts are preserved through the conversion
- Reconstructed object persists correctly via saveRDS to spectra_roundtrip.rds and reloads as a valid Spectra object

## Limitations

- First conversion call takes longer because the reticulate/Python setup must be initialized
- Requires Python >= 3.12 with pinned library versions (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0); if RETICULATE_PYTHON or RETICULATE_PYTHON_ENV is set, all required Python libraries must be installed manually since reticulate will not install them automatically
- Metadata must survive renaming/reformatting across R and Python representations; unmapped or renamed fields can silently mismatch and must be caught by the comparison table

## Evidence

- [methods] conversion functions handle metadata mapping: "contains functions to convert between R's `Spectra::Spectra` objects and `matchms.Spectrum` and `spectrum_utils.spectrum.MsmsSpectrum` objects"
- [abstract] abstract conversion claim: [evidence span withheld — anchor status: not_found]
- [readme] reticulate-based cross-language execution: "create custom functions or workflows on `Spectra` objects in Python
and executing them in R using the *reticulate* R package, and vice versa"
- [readme] reticulate handles Python library installation: "All necessary Python libraries (listed below) are automatically installed by the
[*reticulate*](https://rstudio.github.io/reticulate) R package."
