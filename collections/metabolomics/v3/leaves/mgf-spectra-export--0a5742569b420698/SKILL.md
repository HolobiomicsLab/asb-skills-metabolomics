---
name: mgf-spectra-export
description: Use when a Spectra::Spectra object has been filtered or normalized (e.g.,
  via filterSpectriPy/normalize_intensities) and the resulting peak lists must be
  written out as an MGF file for storage or downstream annotation workflows.
when_to_use_negative:
- Input is raw vendor or mzML data that still needs loading into a Spectra object
  rather than export of already-processed spectra
- The target downstream tool requires a different interchange format (the tutorial
  path uses MGF specifically for matchms/Metabonaut library annotation)
- You only need similarity scoring in-memory (e.g., compareSpectriPy with CosineGreedy)
  and no persistent file output
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SpectriPy
  role: Runs the matchms filtering (filterSpectriPy) whose output is exported to MGF;
    converts spectra between R and Python
  repo: github.com/rformassspectrometry/SpectriPy
- name: Spectra
  role: Holds the MS2 spectra in R (mz/intensity peak lists, precursorMz, msLevel)
    that are written to the MGF file
  repo: github.com/RforMassSpectrometry/Spectra
- name: matchms
  role: Provides the normalize_intensities filter applied before export and the ModifiedCosine
    annotation workflow that consumes MGF spectral libraries
  repo: github.com/matchms
- name: reticulate
  role: Bridges the R Spectra object to Python for the filtering step that precedes
    MGF export (Python >= 3.12, py_require()-managed libraries)
  repo: https://rstudio.github.io/reticulate
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:850c07ecd157edafdf2d3b317e3956dbb1db3cc0533ffa2bd14d31c1247ead9a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Spectra
  - SpectriPy
  - matchms
  - reticulate
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mgf-spectra-export

## Summary

Exports processed/filtered mass spectra (e.g., a Spectra::Spectra object whose intensities were normalized via matchms through SpectriPy) to an MGF peak-list file such as normalized_spectra.mgf, so downstream tools and spectral libraries can consume the processed spectra.

## When to use

Use when a Spectra::Spectra object has been filtered or normalized (e.g., via filterSpectriPy/normalize_intensities) and the resulting peak lists must be written out as an MGF file for storage or downstream annotation workflows.

## When NOT to use

- Input is raw vendor or mzML data that still needs loading into a Spectra object rather than export of already-processed spectra
- The target downstream tool requires a different interchange format (the tutorial path uses MGF specifically for matchms/Metabonaut library annotation)
- You only need similarity scoring in-memory (e.g., compareSpectriPy with CosineGreedy) and no persistent file output

## Inputs

- Spectra::Spectra object containing processed MS2 spectra (e.g., after filterSpectriPy with normalize_intensities)
- spectra variables carried through the R/Python conversion (precursorMz, msLevel, mz and intensity peak lists) via default spectraVariableMapping

## Outputs

- MGF-format peak list file (e.g., normalized_spectra.mgf) with one entry per MS2 spectrum, intensities scaled so the maximum equals 1

## How to apply

After the Python matchms filter has run (filterSpectriPy internally translates the R Spectra::Spectra object to Python matchms.Spectrum objects, executes the filter, and converts results back with pyspec_to_rspec using the default spectraVariableMapping), write the resulting R Spectra object to MGF format. In the source workflow, spectra containing caffeine and 1-methylhistidine MS2 peak lists are normalized so each intensity vector's maximum equals 1, then exported to normalized_spectra.mgf. The same MGF-based output path is used in the package's tutorial ecosystem, where an MGF spectral library is consumed for LC-MS/MS annotation with matchms's ModifiedCosine algorithm, so exporting to MGF keeps processed spectra interoperable with matchms-style library search. Validate the export by re-parsing the MGF and checking that peak m/z and normalized intensity values round-trip correctly.

## Related tools

- **SpectriPy** (Runs the matchms filtering (filterSpectriPy) whose output is exported to MGF; converts spectra between R and Python) — github.com/rformassspectrometry/SpectriPy
- **Spectra** (Holds the MS2 spectra in R (mz/intensity peak lists, precursorMz, msLevel) that are written to the MGF file) — github.com/RforMassSpectrometry/Spectra
- **matchms** (Provides the normalize_intensities filter applied before export and the ModifiedCosine annotation workflow that consumes MGF spectral libraries) — github.com/matchms
- **reticulate** (Bridges the R Spectra object to Python for the filtering step that precedes MGF export (Python >= 3.12, py_require()-managed libraries)) — https://rstudio.github.io/reticulate

## Examples

```
sp_f <- filterSpectriPy(sp, param = normalize_intensities())  ## export the filtered spectra to normalized_spectra.mgf
```

## Evaluation signals

- Every exported spectrum's intensity vector is scaled so its maximum equals 1 — the defining behavior of the matchms normalize_intensities filter — when this skill follows that filter in the workflow
- The MGF file contains the expected number of spectra entries (e.g., the caffeine and 1-methylhistidine MS2 precursor pairs from the example workflow)
- Peak m/z values round-trip unchanged between the Spectra object and the re-parsed MGF (no precision loss in peak positions)
- Retained spectra variables (precursorMz, msLevel) are present in the MGF metadata lines as converted through the default spectraVariableMapping
- The exported MGF can be loaded as a spectral library in the matchms/Metabonaut annotation tutorial path

## Limitations

- Python >= 3.12 is required, and required Python libraries (matchms 0.31, spectrum_utils 0.3.2, numpy 2.2.0) must be installed; the first SpectriPy call is slower because the Python setup has to be initialized
- If RETICULATE_PYTHON or RETICULATE_PYTHON_ENV is set, all required Python libraries must be installed manually, since reticulate will not install them automatically
- The source document's discussion section contains no changelog or extended body content, so detailed parameter documentation for the export step is not available from this context

## Evidence

- [abstract] Applying matchms filters before MGF export: "includes functionality to directly apply spectral similarity, filtering, normalization, etc. routines from the Python matchms library on MS data in R"
- [methods] MGF library usage in the tutorial workflow: "A [tutorial] for the annotation of LC-MS/MS spectra using an MGF library and the ModifiedCosine algorithm from *matchms*."
- [readme] R/Python bridge enabling the filter-then-export workflow: "By wrapping Python functionality into R functions, *SpectriPy* allows a
seamless integration of Python libraries into R."
