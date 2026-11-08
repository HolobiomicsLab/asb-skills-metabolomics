---
name: universal-spectrum-identifier-spectrum-retrieval
description: Use when a workflow needs to fetch a specific public MS/MS spectrum —
  e.g. a ProteomeXchange dataset scan like mzspec:PXD004732:...:scan:41840 or a GNPS
  library entry like mzspec:GNPSLIBRARY:CCMSLIB00000840351 — without downloading the
  source raw files.
when_to_use_negative:
- You already have local spectrum data (mzML, MGF, or in-memory peak arrays) — spectrum_utils
  does not provide raw-file reading functionality and you should use a reader such
  as pymzML or Pyteomics instead.
- You need bulk retrieval of entire datasets or chromatograms rather than individual
  spectra identified by USI.
- You are offline or the hosting resource for the USI is unreachable — from_usi fetches
  spectra over the network and will fail without connectivity.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: spectrum_utils
  role: Provides MsmsSpectrum.from_usi for resolving USIs into in-memory MS/MS spectrum
    objects ready for processing and visualization
  repo: https://github.com/bittremieux/spectrum_utils
- name: pymzML
  role: Alternative external reader for local mzML spectra; spectrum_utils takes data
    from such tools as input instead of reading files itself
  repo: https://github.com/pymzml/pymzML
- name: Pyteomics
  role: External proteomics data-processing toolkit that can supply spectra as input
    when USI retrieval is not used
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_006
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:994146b1486e0f6e51644726d102104409582cd0d57668c169345f4a7e9e47ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Matplotlib
  - Pyteomics
  - Python
  - pymzML
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Universal Spectrum Identifier (USI) spectrum retrieval

## Summary

Retrieves MS/MS spectra from online proteomics and metabolomics resources (ProteomeXchange, PRIDE, GNPS spectral library) by resolving a Universal Spectrum Identifier (USI) via spectrum_utils's MsmsSpectrum.from_usi, eliminating local raw-file handling.

## When to use

Use when a workflow needs to fetch a specific public MS/MS spectrum — e.g. a ProteomeXchange dataset scan like mzspec:PXD004732:...:scan:41840 or a GNPS library entry like mzspec:GNPSLIBRARY:CCMSLIB00000840351 — without downloading the source raw files.

## When NOT to use

- You already have local spectrum data (mzML, MGF, or in-memory peak arrays) — spectrum_utils does not provide raw-file reading functionality and you should use a reader such as pymzML or Pyteomics instead.
- You need bulk retrieval of entire datasets or chromatograms rather than individual spectra identified by USI.
- You are offline or the hosting resource for the USI is unreachable — from_usi fetches spectra over the network and will fail without connectivity.

## Inputs

- Universal Spectrum Identifier string, e.g. mzspec:PXD004732:01650b_BC2-TUM_first_pool_53_01_01-3xHCD-1h-R2:scan:41840
- GNPS spectral-library USI, e.g. mzspec:GNPSLIBRARY:CCMSLIB00000840351
- Network access to the hosting online proteomics/metabolomics data resource

## Outputs

- spectrum_utils.spectrum.MsmsSpectrum object with peak m/z and intensity arrays
- Serialized MsmsSpectrum (e.g. processed_spectrum.pkl via pickle) for later reuse

## How to apply

Construct a USI string of the form mzspec:<dataset accession>:<filename>:<category>:<index> (e.g. scan:41840 for a PXD dataset, or mzspec:GNPSLIBRARY:CCMSLIB00000840351 for a GNPS library entry) and pass it to spectrum_utils.spectrum.MsmsSpectrum.from_usi, which fetches the spectrum over the network. The returned MsmsSpectrum object exposes the peak lists (m/z and intensity arrays) as in-memory NumPy-backed structures ready for downstream processing (set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity) and annotation. Rationale: spectrum_utils deliberately omits its own raw-file readers and instead takes MS data provided by external tools or online resources as input, so USI-based loading is the recommended entry point for quick prototyping. Note that loading requires network access to the hosting resource and that the retrieval mechanism covers both proteomics (ProteomeXchange/PXD) and metabolomics (GNPS library) data resources.

## Related tools

- **spectrum_utils** (Provides MsmsSpectrum.from_usi for resolving USIs into in-memory MS/MS spectrum objects ready for processing and visualization) — https://github.com/bittremieux/spectrum_utils
- **pymzML** (Alternative external reader for local mzML spectra; spectrum_utils takes data from such tools as input instead of reading files itself) — https://github.com/pymzml/pymzML
- **Pyteomics** (External proteomics data-processing toolkit that can supply spectra as input when USI retrieval is not used)

## Examples

```
from spectrum_utils.spectrum import MsmsSpectrum; spectrum = MsmsSpectrum.from_usi('mzspec:PXD004732:01650b_BC2-TUM_first_pool_53_01_01-3xHCD-1h-R2:scan:41840')
```

## Evaluation signals

- The from_usi call returns a MsmsSpectrum object (not None/exception) whose peak list contains the expected number of peaks for the requested scan/library entry.
- The retrieved spectrum's precursor m/z is consistent with the USI's dataset metadata (e.g. the iPRG2012 or PXD004732 scan).
- Downstream processing on the retrieved spectrum applies cleanly: set_mz_range(100, 1400), remove_precursor_peak(10, 'ppm'), filter_intensity(0.05, 50), and scale_intensity('root') all succeed without errors.
- The retrieved-and-processed spectrum serializes successfully to disk (e.g. processed_spectrum.pkl) and reloads, reflecting all applied operations.
- The peak list reflects the correct spectrum: after processing, only peaks in 100–1400 m/z at ≥5% of base-peak intensity (max 50 peaks) remain.

## Limitations

- spectrum_utils does not provide functionality to read spectral data files — it relies on external tools or online USI retrieval, so local raw/mzML files need a separate reader.
- USI retrieval requires network access to the hosting resource (ProteomeXchange, PRIDE, GNPS); unsupported or mistyped USIs will fail to resolve.
- The retrieved spectrum is a single MS/MS spectrum per USI; batch or full-run retrieval is out of scope for this mechanism.

## Evidence

- [readme] Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism.: "Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism."
- [abstract] spectrum_utils is a Python package for mass spectrometry data processing and visualization: "spectrum_utils is a Python package for mass spectrometry data processing and visualization"
- [full_text] Load the spectrum by its USI mzspec:PXD004732:01650b_BC2-TUM_first_pool_53_01_01-3xHCD-1h-R2:scan:41840 using spectrum_utils.spectrum.MsmsSpectrum.from_usi.: [evidence span withheld — anchor status: not_found]
