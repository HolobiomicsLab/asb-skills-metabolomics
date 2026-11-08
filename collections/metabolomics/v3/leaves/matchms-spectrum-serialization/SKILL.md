---
name: matchms-spectrum-serialization
description: Use when R Spectra::Spectra data has been translated to Python matchms.Spectrum
  objects via SpectriPy/reticulate and you must persist the converted spectra and
  their metadata mapping for downstream Python processing or round-trip validation.
when_to_use_negative:
- The data is still on the R side as Spectra::Spectra and no rspec_to_pyspec() conversion
  has been performed — use the SpectriPy conversion skill first, since there are no
  Python matchms.Spectrum objects to serialize.
- The goal is similarity scoring or filtering in R via compareSpectriPy() or filterSpectriPy()
  — SpectriPy wraps Python matchms routines into R functions directly, so no serialization
  step is needed.
- The target format is MGF or another interchange file rather than Python-native persistence
  — pickle round-trips only matchms objects, not general text interchange formats.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SpectriPy
  role: Provides rspec_to_pyspec() to translate R Spectra::Spectra into matchms.Spectrum
    objects, renaming/reformatting metadata per spectraVariableMapping before serialization
  repo: https://github.com/rformassspectrometry/SpectriPy
- name: matchms
  role: Python library whose matchms.Spectrum objects are the serialization payload;
    used to reload and validate the pickle (tested version 0.31)
  repo: https://github.com/matchms
- name: reticulate
  role: R package that initializes the Python environment and manages the Python library
    environment (via py_require()) under which the conversion and serialization run
  repo: https://rstudio.github.io/reticulate
- name: Spectra
  role: Source of the R Spectra::Spectra objects (e.g. the caffeine and 1-methylhistidine
    MS2 vignette spectra) that feed the conversion
  repo: https://github.com/RforMassSpectrometry/Spectra
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:7c68456452e4397800f50fe5876003ae74d5c23203e33965ce8089c349a333e9
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# matchms-spectrum-serialization

## Summary

Serializes converted matchms.Spectrum objects (produced by SpectriPy's rspec_to_pyspec() translation of R Spectra::Spectra objects) to a pickle file, together with the applied spectra variable mapping, so the cross-language conversion can be validated and reused in Python matchms workflows.

## When to use

Use when R Spectra::Spectra data has been translated to Python matchms.Spectrum objects via SpectriPy/reticulate and you must persist the converted spectra and their metadata mapping for downstream Python processing or round-trip validation.

## When NOT to use

- The data is still on the R side as Spectra::Spectra and no rspec_to_pyspec() conversion has been performed — use the SpectriPy conversion skill first, since there are no Python matchms.Spectrum objects to serialize.
- The goal is similarity scoring or filtering in R via compareSpectriPy() or filterSpectriPy() — SpectriPy wraps Python matchms routines into R functions directly, so no serialization step is needed.
- The target format is MGF or another interchange file rather than Python-native persistence — pickle round-trips only matchms objects, not general text interchange formats.

## Inputs

- List of matchms.Spectrum objects produced by rspec_to_pyspec() conversion of an R Spectra::Spectra object
- Spectra variable mapping (spectraVariableMapping / defaultSpectraVariableMapping) defining R column name to matchms metadata key renames
- Source R Spectra metadata (msLevel, precursorMz, retention time, name, id) and peak m/z/intensity arrays

## Outputs

- Pickle file containing the list of matchms.Spectrum objects
- Tabular CSV file recording the applied metadata variable mapping (R column name vs. matchms metadata key)

## How to apply

After running rspec_to_pyspec() on an R Spectra object (with the pythonLibrary argument set to 'matchms'), the metadata columns — MS level, precursorMz, retention time, and compound name/id — have been renamed and reformatted according to the defaultSpectraVariableMapping / spectraVariableMapping definitions to comply with matchms metadata conventions. Serialize the resulting list of matchms.Spectrum objects to a pickle file, and write the applied metadata variable mapping (R column name vs. matchms metadata key) to a tabular CSV file so the renaming is auditable. To validate, reload the pickle in Python with matchms and confirm each Spectrum reproduces the source spectra's peak m/z and intensity arrays and that every mapped metadata field is present under its renamed key (e.g. msLevel, precursorMz, name, id from the caffeine and 1-methylhistidine MS2 example). Pickling is the mechanism because SpectriPy's translation crosses the reticulate boundary, so the Python-side objects must be captured in a Python-native serialization format to be used independently of the R session.

## Related tools

- **SpectriPy** (Provides rspec_to_pyspec() to translate R Spectra::Spectra into matchms.Spectrum objects, renaming/reformatting metadata per spectraVariableMapping before serialization) — https://github.com/rformassspectrometry/SpectriPy
- **matchms** (Python library whose matchms.Spectrum objects are the serialization payload; used to reload and validate the pickle (tested version 0.31)) — https://github.com/matchms
- **reticulate** (R package that initializes the Python environment and manages the Python library environment (via py_require()) under which the conversion and serialization run) — https://rstudio.github.io/reticulate
- **Spectra** (Source of the R Spectra::Spectra objects (e.g. the caffeine and 1-methylhistidine MS2 vignette spectra) that feed the conversion) — https://github.com/RforMassSpectrometry/Spectra

## Examples

```
py_spec <- rspec_to_pyspec(sps, pythonLibrary = "matchms"); reticulate::py_save_object(py_spec, "spectra.pkl")
```

## Evaluation signals

- Reload the pickle in Python and confirm each matchms.Spectrum reproduces the source spectra's peak m/z and intensity arrays exactly.
- Check that every mapped metadata field (MS level, precursor m/z, retention time, compound name/id) is present in the reloaded Spectrum under its renamed matchms key per the spectraVariableMapping.
- Confirm the mapping CSV lists a row for each applied R column name vs. matchms metadata key, matching the defaultSpectraVariableMapping actually used in the conversion.
- Spectra count in the pickle equals the number of spectra in the source R Spectra object (e.g. two MS2 spectra each for caffeine and 1-methylhistidine).

## Limitations

- SpectriPy requires Python >= 3.12 with tested versions matchms 0.31, spectrum_utils 0.3.2, and numpy 2.2.0; if RETICULATE_PYTHON or RETICULATE_PYTHON_ENV variables are set, all required Python libraries must be manually installed since reticulate will not install them automatically.
- The first SpectriPy/reticulate call takes longer because the Python setup has to be initialized.
- Metadata renaming/reformatting is governed by the spectra variable mapping; arbitrary metadata not covered by the mapping must be handled explicitly or it will not round-trip under matchms keys.

## Evidence

- [other] SpectriPy's translation mechanism handles conversion between the R and Python MS data structures including any required renaming and reformatting of spectra metadata, such as MS level and retention times: [evidence span withheld — anchor status: not_found]
- [abstract] It can convert between R's Spectra::Spectra and Python's matchms.Spectrum and spectrum_utils.spectrum.MsmsSpectrum objects: [evidence span withheld — anchor status: not_found]
- [readme] R and Python spectral objects are easily translated and available in one workflow, with custom Python workflows executed in R via reticulate: "R and Python spectral objects are easily translated and available
in one workflow (i.e., a quarto document), enabling the advanced user or developer to create custom functions or workflows on"
- [readme] Required Python libraries are automatically installed by reticulate; tested versions are matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0: "All
necessary Python libraries (listed below) are automatically installed by the
[*reticulate*](https://rstudio.github.io/reticulate) R package."
