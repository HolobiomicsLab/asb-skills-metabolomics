---
name: spectral-intensity-normalization
description: Use when MS2 peak intensities in an R Spectra::Spectra object are on
  arbitrary raw scales and need unit-max normalization (e.g. before cosine similarity
  scoring or cross-dataset comparison) using matchms routines from R.
when_to_use_negative:
- Intensities are already scaled (max = 1) or the downstream algorithm requires raw
  intensities — normalization would distort quantitative information
- The desired operation is peak filtering rather than scaling — SpectriPy instead
  exposes remove_peaks_around_precursor_mz(mz_tolerance), select_by_intensity(intensity_from,
  intensity_to), and select_by_mz(mz_from, mz_to)
- The analysis is entirely in Python with matchms directly — no R-to-Python translation
  via SpectriPy is needed
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SpectriPy
  role: Provides filterSpectriPy() and the normalize_intensities parameter object
    that wrap matchms filtering on Spectra objects; handles R<->Python conversion
  repo: https://github.com/RforMassSpectrometry/SpectriPy
- name: matchms
  role: Python library whose normalize_intensities filter is executed on the translated
    spectrum objects
  repo: https://github.com/matchms
- name: Spectra
  role: R infrastructure holding the MS2 spectra before and after filtering
  repo: https://github.com/RforMassSpectrometry/Spectra
- name: reticulate
  role: R package handling Python configuration, library installation via py_require(),
    and execution of the Python filter
  repo: https://rstudio.github.io/reticulate
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:b0ec96c60431333d57cc021af327ba51f2f190051e769b6dd2c7da740bb760c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RforMassSpectrometry/SpectriPy
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
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# spectral-intensity-normalization

## Summary

Applies the matchms normalize_intensities filter to MS2 spectra held in an R Spectra::Spectra object via SpectriPy's filterSpectriPy(), scaling each spectrum's peak intensities so the maximum equals 1 by translating data to Python, running the filter, and converting results back to R.

## When to use

Use when MS2 peak intensities in an R Spectra::Spectra object are on arbitrary raw scales and need unit-max normalization (e.g. before cosine similarity scoring or cross-dataset comparison) using matchms routines from R.

## When NOT to use

- Intensities are already scaled (max = 1) or the downstream algorithm requires raw intensities — normalization would distort quantitative information
- The desired operation is peak filtering rather than scaling — SpectriPy instead exposes remove_peaks_around_precursor_mz(mz_tolerance), select_by_intensity(intensity_from, intensity_to), and select_by_mz(mz_from, mz_to)
- The analysis is entirely in Python with matchms directly — no R-to-Python translation via SpectriPy is needed

## Inputs

- Spectra::Spectra R object with MS2 spectra (mz and intensity peak lists, precursorMz)
- SpectriPy normalize_intensities filter parameter object

## Outputs

- Spectra::Spectra object with intensities normalized to unit maximum
- Optionally an MGF file of normalized spectra (e.g. normalized_spectra.mgf)

## How to apply

Load the Spectra and SpectriPy packages in R (reticulate handles the Python >= 3.12 configuration via py_require()). Build or load a Spectra::Spectra object containing the MS2 spectra (e.g. caffeine and 1-methylhistidine precursor pairs), then construct SpectriPy's normalize_intensities filter parameter object and invoke filterSpectriPy() on the Spectra object. SpectriPy internally translates the R Spectra data to Python matchms.Spectrum objects, executes matchms's normalize_intensities filter, and converts the results back to R data types. The processed spectra can be retrieved explicitly with pyspec_to_rspec() using the default spectraVariableMapping and exported to MGF (e.g. normalized_spectra.mgf). Validate that every output spectrum's intensity vector is scaled so its maximum equals 1, which is the defining behavior of normalize_intensities. Note the first SpectriPy call is slower because the Python setup must be initialized.

## Related tools

- **SpectriPy** (Provides filterSpectriPy() and the normalize_intensities parameter object that wrap matchms filtering on Spectra objects; handles R<->Python conversion) — https://github.com/RforMassSpectrometry/SpectriPy
- **matchms** (Python library whose normalize_intensities filter is executed on the translated spectrum objects) — https://github.com/matchms
- **Spectra** (R infrastructure holding the MS2 spectra before and after filtering) — https://github.com/RforMassSpectrometry/Spectra
- **reticulate** (R package handling Python configuration, library installation via py_require(), and execution of the Python filter) — https://rstudio.github.io/reticulate

## Examples

```
library(Spectra); library(SpectriPy); sps_norm <- filterSpectriPy(sps, param = normalize_intensities())
```

## Evaluation signals

- In every normalized spectrum, the intensity vector's maximum equals 1 (defining behavior of matchms normalize_intensities)
- Number of spectra and number of peaks per spectrum are unchanged by normalization (only intensity scale changes, mz values untouched)
- round-trip consistency: pyspec_to_rspec() with default spectraVariableMapping returns spectra variables matching the original Spectra object
- Exported normalized_spectra.mgf parses back to unit-max intensity vectors
- First filterSpectriPy() call initializes Python; subsequent calls should run faster — failures usually indicate reticulate/Python configuration problems

## Limitations

- Requires Python >= 3.12; if RETICULATE_PYTHON or RETICULATE_PYTHON_ENV environment variables are set, all required Python libraries (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0) must be installed manually since reticulate will not install them automatically
- First SpectriPy call takes longer because the Python setup has to be initialized
- Normalization discards absolute intensity scale information, which may be needed for quantitative workflows

## Evidence

- [methods] filterSpectriPy mechanism: "includes functionality to directly apply spectral similarity, filtering, normalization, etc. routines from the Python matchms library on MS data in R"
- [methods] conversion mechanism: "contains functions to convert between R's `Spectra::Spectra` objects and `matchms.Spectrum` and `spectrum_utils.spectrum.MsmsSpectrum` objects"
- [methods] filter list: "- `normalize_intensities`"
- [readme] python config: "*SpectriPy* needs Python (version >= 3.12) to be installed on the system. All
necessary Python libraries (listed below) are automatically installed by"
- [readme] tested versions: "- [*matchms*](https://github.com/matchms) 0.31
- [*spectrum_utils*](https://github.com/bittremieux-lab/spectrum_utils) 0.3.2
- *numpy* 2.2.0"
- [readme] first call latency: "Note also that the first `compareSpectriPy()` call takes usually a little longer
because the Python setup has to be initialized."
