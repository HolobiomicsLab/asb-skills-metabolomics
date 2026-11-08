---
name: spectra-metadata-variable-mapping
description: Use when translating R Spectra::Spectra objects to Python matchms.Spectrum
  objects and spectra metadata (MS level, precursor m/z, retention times, arbitrary
  annotations) must be renamed/reformatted to matchms conventions.
when_to_use_negative:
- When the goal is similarity scoring only — compareSpectriPy() handles conversion
  internally and returns a numeric similarity matrix, so manual variable mapping is
  unnecessary
- When converting in the opposite direction (Python to R) or to spectrum_utils.spectrum.MsmsSpectrum
  objects — a different SpectriPy conversion function and mapping applies
- When the R object is not a Spectra::Spectra object (e.g. a raw mzML file) — import
  it into Spectra first via the appropriate backend
tools:
- name: SpectriPy
  role: Provides rspec_to_pyspec(), defaultSpectraVariableMapping, and the reticulate-based
    R-to-Python translation of spectra metadata
  repo: https://github.com/rformassspectrometry/SpectriPy
- name: Spectra
  role: Supplies the source R Spectra::Spectra object and its spectra variables that
    are remapped
  repo: https://github.com/RforMassSpectrometry/Spectra
- name: matchms
  role: Defines the target metadata conventions; converted objects become matchms.Spectrum
    instances
  repo: https://github.com/matchms
- name: reticulate
  role: R package used to initialize and bridge the Python environment in which the
    converted Spectrum objects live
  repo: https://rstudio.github.io/reticulate
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:11136c3363d5338ea44d1635b77a67811e4db1cce9c0da3af652a09ed3eae1f6
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spectra-metadata-variable-mapping

## Summary

Maps R Spectra::Spectra spectra variables (msLevel, precursorMz, retention time, compound name/ID) onto matchms metadata keys via SpectriPy's defaultSpectraVariableMapping during rspec_to_pyspec() conversion, so metadata survives the R-to-Python translation under matchms naming conventions.

## When to use

Use when translating R Spectra::Spectra objects to Python matchms.Spectrum objects and spectra metadata (MS level, precursor m/z, retention times, arbitrary annotations) must be renamed/reformatted to matchms conventions.

## When NOT to use

- When the goal is similarity scoring only — compareSpectriPy() handles conversion internally and returns a numeric similarity matrix, so manual variable mapping is unnecessary
- When converting in the opposite direction (Python to R) or to spectrum_utils.spectrum.MsmsSpectrum objects — a different SpectriPy conversion function and mapping applies
- When the R object is not a Spectra::Spectra object (e.g. a raw mzML file) — import it into Spectra first via the appropriate backend

## Inputs

- R Spectra::Spectra object (e.g. built from a DataFrame with msLevel, precursorMz, name, id columns plus mz and intensity peak lists)
- spectraVariableMapping / defaultSpectraVariableMapping definition (R column name vs matchms metadata key)
- pythonLibrary argument selecting the target Python class (e.g. 'matchms')

## Outputs

- List of Python matchms.Spectrum objects with peaks (m/z, intensity arrays) and renamed metadata keys
- Optional tabular CSV of the applied metadata variable mapping (R column name vs matchms metadata key)

## How to apply

Initialize the Python environment through reticulate (SpectriPy uses py_require() from reticulate >= 1.41 and requires matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0). Call rspec_to_pyspec() on the R Spectra object with the pythonLibrary argument set to 'matchms'. The conversion consults the defaultSpectraVariableMapping (or a user-supplied spectraVariableMapping) to rename and reformat each spectra variable: R-style columns such as msLevel, precursorMz, rtime, name and id are mapped to the corresponding matchms metadata keys (e.g. mslevel, precursor_mz, retention_time, compound_name). Peaks (m/z and intensity arrays) are transferred as numeric arrays without remapping; only metadata keys are translated. Unmapped arbitrary metadata is carried over per the mapping definition, and the applied R-column-to-matchms-key mapping can be exported as a tabular CSV for provenance. The decision point is whether to accept defaultSpectraVariableMapping or override it when custom spectra variables exist in the source Spectra object.

## Related tools

- **SpectriPy** (Provides rspec_to_pyspec(), defaultSpectraVariableMapping, and the reticulate-based R-to-Python translation of spectra metadata) — https://github.com/rformassspectrometry/SpectriPy
- **Spectra** (Supplies the source R Spectra::Spectra object and its spectra variables that are remapped) — https://github.com/RforMassSpectrometry/Spectra
- **matchms** (Defines the target metadata conventions; converted objects become matchms.Spectrum instances) — https://github.com/matchms
- **reticulate** (R package used to initialize and bridge the Python environment in which the converted Spectrum objects live) — https://rstudio.github.io/reticulate

## Examples

```
pyspec <- rspec_to_pyspec(sps, pythonLibrary = "matchms")
```

## Evaluation signals

- Reload the serialized pickle in Python and confirm each matchms.Spectrum reproduces the source spectra's peak m/z and intensity arrays
- Verify every mapped metadata field (msLevel, precursorMz, retention time, name, id) is present under its renamed matchms key
- Check that the exported CSV of the variable mapping lists an R column name and matchms key for every transferred metadata variable
- Confirm round-trip consistency: converting back (pyspec_to_rspec) recovers the original spectra variable names
- First conversion call is slower due to Python setup initialization via reticulate; subsequent calls should not re-initialize

## Limitations

- SpectriPy requires Python >= 3.12 and specific tested Python library versions (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0); if RETICULATE_PYTHON or RETICULATE_PYTHON_ENV are set, all listed libraries must be pre-installed manually
- Custom spectra variables not covered by the mapping definition may not transfer unless spectraVariableMapping is overridden
- The first SpectriPy call takes longer because the Python setup has to be initialized

## Evidence

- [abstract] Translation of spectra metadata: "provides functionality to efficiently translate between R and Python MS data structures"
- [methods] Conversion function scope: "contains functions to convert between R's `Spectra::Spectra` objects and `matchms.Spectrum` and `spectrum_utils.spectrum.MsmsSpectrum` objects"
- [readme] Package role: "R and Python spectral objects are easily translated and available in one workflow (i.e., a quarto document)"
- [readme] Tested Python versions: [evidence span withheld — anchor status: not_found]
