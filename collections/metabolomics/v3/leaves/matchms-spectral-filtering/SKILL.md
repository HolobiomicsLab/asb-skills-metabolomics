---
name: matchms-spectral-filtering
description: Use when MS2 peak lists held in an R Spectra::Spectra object need matchms
  processing steps such as intensity normalization or m/z/intensity window selection
  without leaving the R workflow.
when_to_use_negative:
- The task is similarity scoring rather than filtering/normalization — use compareSpectriPy()
  with CosineGreedyParam or ModifiedCosine instead.
- The spectra already live in Python as matchms.Spectrum or spectrum_utils.spectrum.MsmsSpectrum
  objects — apply matchms filters directly in Python without the R↔Python translation
  overhead.
- A pure-R processing step (e.g. Spectra/MsCoreUtils built-ins) suffices and no matchms-specific
  filter is required.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SpectriPy
  role: Provides filterSpectriPy() and the matchms filter parameter objects; converts
    R Spectra objects to Python and back
  repo: https://github.com/rformassspectrometry/SpectriPy
- name: matchms
  role: Python library whose filtering/normalization routines (e.g. normalize_intensities)
    are executed on the translated spectra
  repo: https://github.com/matchms
- name: Spectra
  role: R package defining the Spectra::Spectra container that holds the MS2 peak
    data being filtered
  repo: https://github.com/RforMassSpectrometry/Spectra
- name: reticulate
  role: R package handling the Python configuration and R↔Python data translation
    during filterSpectriPy() execution
  repo: https://rstudio.github.io/reticulate
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:30e6dcc5b1b99172c48afafb36f92853c2fe98f9957d92b25e967e989e1396f7
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# matchms-spectral-filtering

## Summary

Apply Python matchms filtering routines (e.g. normalize_intensities, select_by_mz, remove_peaks_around_precursor_mz) to R Spectra::Spectra objects via SpectriPy's filterSpectriPy(), which translates MS data R→Python→R using reticulate.

## When to use

Use when MS2 peak lists held in an R Spectra::Spectra object need matchms processing steps such as intensity normalization or m/z/intensity window selection without leaving the R workflow.

## When NOT to use

- The task is similarity scoring rather than filtering/normalization — use compareSpectriPy() with CosineGreedyParam or ModifiedCosine instead.
- The spectra already live in Python as matchms.Spectrum or spectrum_utils.spectrum.MsmsSpectrum objects — apply matchms filters directly in Python without the R↔Python translation overhead.
- A pure-R processing step (e.g. Spectra/MsCoreUtils built-ins) suffices and no matchms-specific filter is required.

## Inputs

- Spectra::Spectra object containing MS2 spectra (e.g. built from a DataFrame with msLevel, precursorMz, mz and intensity list-columns)
- SpectriPy filter parameter object (normalize_intensities, remove_peaks_around_precursor_mz, select_by_intensity, select_by_mz)

## Outputs

- Filtered Spectra::Spectra object with peak intensity vectors rescaled so the maximum equals 1 (for normalize_intensities), or peaks removed per the selected filter criteria
- Optional exported MGF file of the normalized spectra (e.g. normalized_spectra.mgf)

## How to apply

Load the Spectra and SpectriPy packages in R (reticulate handles the Python >= 3.12 configuration) and build a Spectra::Spectra object from the MS2 spectra of interest, e.g. caffeine and 1-methylhistidine precursor pairs. Construct the matchms filter parameter object exported by SpectriPy — normalize_intensities (no arguments), remove_peaks_around_precursor_mz(mz_tolerance), select_by_intensity(intensity_from, intensity_to), or select_by_mz(mz_from, mz_to) — and call filterSpectriPy() on the Spectra object so the filter executes on the converted Python matchms.Spectrum objects. Retrieve processed spectra back to R with pyspec_to_rspec() using the default spectraVariableMapping, then export (e.g. to MGF) if needed. Rationale: SpectriPy wraps Python matchms functionality into R functions so filtering, normalization and similarity routines run on native Python spectrum objects without manual data munging.

## Related tools

- **SpectriPy** (Provides filterSpectriPy() and the matchms filter parameter objects; converts R Spectra objects to Python and back) — https://github.com/rformassspectrometry/SpectriPy
- **matchms** (Python library whose filtering/normalization routines (e.g. normalize_intensities) are executed on the translated spectra) — https://github.com/matchms
- **Spectra** (R package defining the Spectra::Spectra container that holds the MS2 peak data being filtered) — https://github.com/RforMassSpectrometry/Spectra
- **reticulate** (R package handling the Python configuration and R↔Python data translation during filterSpectriPy() execution) — https://rstudio.github.io/reticulate

## Examples

```
library(Spectra); library(SpectriPy); s <- filterSpectriPy(s, param = normalize_intensities()); export(s, MsBackendMgf(), "normalized_spectra.mgf")
```

## Evaluation signals

- After normalize_intensities, every output spectrum's intensity vector has a maximum of exactly 1 (the defining behavior of the matchms normalize_intensities filter).
- Number of peaks per spectrum only decreases (or stays equal) after select_by_mz / select_by_intensity / remove_peaks_around_precursor_mz, and retained peaks fall inside the chosen m/z or intensity window.
- Spectra metadata (precursorMz, msLevel, name/id spectra variables) is preserved after the round trip through pyspec_to_rspec() with the default spectraVariableMapping.
- The exported normalized_spectra.mgf parses and contains one entry per input spectrum with matching precursor m/z values.

## Limitations

- Requires Python >= 3.12 and reticulate >= 1.41 (py_require() library management); if RETICULATE_PYTHON or RETICULATE_PYTHON_ENV is set, all required Python libraries (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0) must be installed manually.
- The first call that initializes the Python setup takes noticeably longer than subsequent calls.
- Conversion currently targets matchms.Spectrum and spectrum_utils.spectrum.MsmsSpectrum objects only; other Python MS data structures are not covered.

## Evidence

- [abstract] filterSpectriPy translates R→Python→R: "provides functionality to efficiently translate between R and Python MS data structures"
- [abstract] filterSpectriPy applies matchms filtering on R MS data: "includes functionality to directly apply spectral similarity, filtering, normalization, etc. routines from the Python matchms library on MS data in R"
- [methods] Filter steps enumerated in the methods: "- `normalize_intensities`
- `remove_peaks_around_precursor_mz(mz_tolerance)`
- `select_by_intensity(intensity_from, intensity_to)`
- `select_by_mz(mz_from, mz_to)`
- `filterSpectriPy`"
- [readme] Python version requirement: "*SpectriPy* needs Python (version >= 3.12) to be installed on the system. All
necessary Python libraries (listed below) are automatically installed by"
- [methods] Python round trip and reticulate usage for custom workflows: "create custom functions or workflows on `Spectra` objects in Python
and executing them in R using the *reticulate* R package, and vice versa"
