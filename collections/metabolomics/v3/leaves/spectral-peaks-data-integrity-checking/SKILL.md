---
name: spectral-peaks-data-integrity-checking
description: Use when a workflow has converted spectra objects across the R/Python
  boundary (rspec_to_pyspec followed by pyspec_to_rspec) and the reconstructed object
  must be validated before downstream use.
when_to_use_negative:
- When the spectra objects have not crossed the R/Python conversion boundary (e.g.
  both are native R Spectra objects) — a direct diff suffices
- When validating Python-side matchms-only processing (e.g. filterSpectriPy filtering
  or compareSpectriPy scoring) rather than a structural round-trip conversion
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SpectriPy
  role: Provides rspec_to_pyspec() and pyspec_to_rspec() conversion functions whose
    output is being integrity-checked; handles renaming and reformatting of spectra
    metadata such as MS level and retention times
  repo: github.com/rformassspectrometry/SpectriPy
- name: Spectra
  role: Defines the Spectra::Spectra object whose peaks data (mz/intensity matrices)
    and spectra variables are compared
  repo: github.com/RforMassSpectrometry/Spectra
- name: reticulate
  role: Bridges the Python matchms.Spectrum objects into the R session for the round-trip
    comparison
  repo: https://rstudio.github.io/reticulate
- name: matchms
  role: Python library whose Spectrum objects are the intermediate representation
    being validated; invoked via pythonLibrary='matchms'
  repo: github.com/matchms
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:bb8d69d828f9e7f74ddd1e5a6b1d3500d3edf6fc41e97f569421071c58f7edf6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Spectra
  - SpectriPy
  - matchms
  - reticulate
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Spectral peaks data integrity checking

## Summary

Verifies that a round-trip conversion between R Spectra::Spectra and Python matchms.Spectrum objects preserves peak data and metadata by comparing mz/intensity matrices and mapped spectra variables element-wise against the original object.

## When to use

Use when a workflow has converted spectra objects across the R/Python boundary (rspec_to_pyspec followed by pyspec_to_rspec) and the reconstructed object must be validated before downstream use.

## When NOT to use

- When the spectra objects have not crossed the R/Python conversion boundary (e.g. both are native R Spectra objects) — a direct diff suffices
- When validating Python-side matchms-only processing (e.g. filterSpectriPy filtering or compareSpectriPy scoring) rather than a structural round-trip conversion

## Inputs

- Original R Spectra::Spectra object
- Round-tripped Spectra::Spectra object reconstructed from Python matchms.Spectrum objects via pyspec_to_rspec()
- Mapped spectra variables to check (MS level, precursorMz, retention times, compound name)

## Outputs

- Boolean/passed-flag round-trip validation result
- Metadata comparison table with per-variable original value, round-trip value, and match flag
- Serialized reconstructed Spectra object and comparison table (spectra_roundtrip.rds)

## How to apply

After loading the matchms.Spectrum objects back into R via reticulate and rebuilding a Spectra object with pyspec_to_rspec() (using defaultSpectraVariableMapping() and pythonLibrary='matchms'), compare the peaks data (mz and intensity matrices) against the original Spectra object using all.equal with standard floating-point tolerance. Then compare mapped spectra variables (MS level, precursorMz, retention times, compound name) element-wise between the original and reconstructed objects, recording any mismatches. Assemble a metadata comparison table listing each spectra variable with its original value, round-tripped value, and a match flag. Accept the round trip only if all mz/intensity pairs match within numeric tolerance and every mapped metadata field is identical; save the reconstructed Spectra object and the comparison table (e.g. spectra_roundtrip.rds via saveRDS) for auditing.

## Related tools

- **SpectriPy** (Provides rspec_to_pyspec() and pyspec_to_rspec() conversion functions whose output is being integrity-checked; handles renaming and reformatting of spectra metadata such as MS level and retention times) — github.com/rformassspectrometry/SpectriPy
- **Spectra** (Defines the Spectra::Spectra object whose peaks data (mz/intensity matrices) and spectra variables are compared) — github.com/RforMassSpectrometry/Spectra
- **reticulate** (Bridges the Python matchms.Spectrum objects into the R session for the round-trip comparison) — https://rstudio.github.io/reticulate
- **matchms** (Python library whose Spectrum objects are the intermediate representation being validated; invoked via pythonLibrary='matchms') — github.com/matchms

## Examples

```
r_spec <- SpectriPy::pyspec_to_rspec(py_spectra, defaultSpectraVariableMapping(), pythonLibrary = "matchms"); stopifnot(isTRUE(all.equal(peaksData(r_spec), peaksData(orig))))
```

## Evaluation signals

- all.equal on mz and intensity matrices of original vs. round-tripped Spectra returns TRUE within standard floating-point tolerance
- Every mapped spectra variable (MS level, precursorMz, retention times, compound name) is identical element-wise between original and reconstructed objects
- Metadata comparison table shows a TRUE match flag for every listed spectra variable
- Number of spectra in the reconstructed Spectra object equals the number in the original object
- Reconstructed object and comparison table are successfully serialized to spectra_roundtrip.rds

## Limitations

- Floating-point tolerance in all.equal means exact bitwise identity of mz/intensity values is not guaranteed across the conversion boundary
- Only variables covered by defaultSpectraVariableMapping() are checked; unmapped spectra variables may be silently dropped or renamed by the conversion
- First conversion/reticulate call incurs Python setup initialization overhead

## Evidence

- [methods] SpectriPy's conversion functions preserve peaks data and metadata: [evidence span withheld — anchor status: not_found]
- [methods] Conversion between R and Python spectrum objects is a supported workflow step: "contains
functions to convert between R's `Spectra::Spectra` objects and
`matchms.Spectrum` and `spectrum_utils.spectrum.MsmsSpectrum` objects"
- [methods] reticulate enables custom Python workflows on Spectra objects in R: "create custom functions or workflows on `Spectra` objects in Python
and executing them in R using the *reticulate* R package, and vice versa"
- [abstract] SpectriPy translates between R and Python MS data structures: "provides functionality to efficiently translate between R and Python MS data structures"
