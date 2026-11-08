---
name: precursor-peak-removal
description: Use when a USI-loaded or MGF-loaded MS/MS spectrum (e.g. mzspec:PXD004732:...:scan:41840
  or iPRG2012.mgf) still contains the precursor ion peak and must be cleaned before
  fragment annotation, noise filtering, or plotting.
when_to_use_negative:
- Input is raw spectral data still in file form (mzML/MGF) — spectrum_utils does not
  provide functionality to read spectral data files and relies on external tools such
  as pymzML or Pyteomics for this task
- The precursor ion signal itself is the target of the analysis (e.g. intact precursor
  characterization) rather than the fragment peaks
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: spectrum_utils
  role: Provides the remove_precursor_peak(fragment_tol_mass, fragment_tol_mode) processing
    method applied to MsmsSpectrum objects as part of the standard processing chain
  repo: https://github.com/bittremieux/spectrum_utils
- name: pymzML
  role: Alternative/external tool for reading mzML spectral data, since spectrum_utils
    itself does not read spectral data files
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:d419a81b97a285a5c53f128db97a84ebb695af2992e941634b81705916debecd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Pyteomics
  - Python
  - pymzML
  - spectrum_utils
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# precursor-peak-removal

## Summary

Removes the precursor ion peak from an MS/MS spectrum loaded into spectrum_utils, so that downstream processing, fragment annotation (a/b/y ions), and visualization focus on fragment peaks rather than the intact precursor. It is one of the standard spectrum processing operations the package provides.

## When to use

Use when a USI-loaded or MGF-loaded MS/MS spectrum (e.g. mzspec:PXD004732:...:scan:41840 or iPRG2012.mgf) still contains the precursor ion peak and must be cleaned before fragment annotation, noise filtering, or plotting.

## When NOT to use

- Input is raw spectral data still in file form (mzML/MGF) — spectrum_utils does not provide functionality to read spectral data files and relies on external tools such as pymzML or Pyteomics for this task
- The precursor ion signal itself is the target of the analysis (e.g. intact precursor characterization) rather than the fragment peaks

## Inputs

- A spectrum_utils MsmsSpectrum object loaded via MsmsSpectrum.from_usi (e.g. mzspec:PXD004732:01650b_BC2-TUM_first_pool_53_01_01-3xHCD-1h-R2:scan:41840) or read from an MGF file (e.g. iPRG2012.mgf)
- Precursor m/z and charge metadata available on the spectrum object so the precursor peak can be located
- A precursor mass tolerance specification (10 ppm used in the paper's pipeline)

## Outputs

- The same MsmsSpectrum object with the precursor ion peak removed from its peak list, ready for filter_intensity, scale_intensity, and annotate_proforma steps
- Optionally a serialized processed spectrum (processed_spectrum.pkl) reflecting all applied processing operations

## How to apply

Load the spectrum as a spectrum_utils MsmsSpectrum (via MsmsSpectrum.from_usi with a Universal Spectrum Identifier, or from an MGF such as iPRG2012.mgf), then call remove_precursor_peak(fragment_tol_mass=10, fragment_tol_mode='ppm') to remove the precursor ion peak, matching it within a 10 ppm tolerance. In the canonical pipeline from the paper this step is applied after restricting the m/z range to 100–1400 m/z with set_mz_range and before filtering low-intensity noise peaks with filter_intensity(min_intensity=0.05, max_num_peaks=50) and root-scaling intensities with scale_intensity('root'). The rationale is that the precursor peak dominates the spectrum but carries no fragment information, so its removal prevents it from overwhelming noise filtering, intensity scaling, and ProForma-based fragment annotation (annotate_proforma with ion_types='aby'). Keep the precursor mass tolerance in ppm mode so the tolerance scales with the precursor m/z rather than being a fixed Da window.

## Related tools

- **spectrum_utils** (Provides the remove_precursor_peak(fragment_tol_mass, fragment_tol_mode) processing method applied to MsmsSpectrum objects as part of the standard processing chain) — https://github.com/bittremieux/spectrum_utils
- **pymzML** (Alternative/external tool for reading mzML spectral data, since spectrum_utils itself does not read spectral data files) — https://github.com/pymzml/pymzML

## Examples

```
spec = sus.MsmsSpectrum.from_usi(usi).set_mz_range(min_mz=100, max_mz=1400).remove_precursor_peak(fragment_tol_mass=10, fragment_tol_mode='ppm')
```

## Evaluation signals

- After remove_precursor_peak(fragment_tol_mass=10, fragment_tol_mode='ppm'), no peak within the 10 ppm window around the precursor m/z remains in the spectrum's peak list
- The removal step occurs in the correct pipeline order: after set_mz_range(100–1400 m/z) and before filter_intensity(min_intensity=0.05, max_num_peaks=50)
- Peak counts and m/z values of the processed spectrum differ from the unprocessed spectrum only by the removal of low-value peaks, not by spurious fragment loss near the precursor
- The processed MsmsSpectrum serializes and reloads successfully (e.g. from processed_spectrum.pkl) with the precursor peak absent

## Limitations

- spectrum_utils does not provide functionality to read spectral data files; the spectrum must first be loaded with an external tool such as pymzML or Pyteomics or via a USI
- The removal tolerance is mass-based (10 ppm in the demonstrated workflow), so very small precursor mass errors outside the tolerance window would leave residual precursor signal
- Precursor removal is only one step of the processing chain — it does not by itself remove noise peaks (that requires filter_intensity) or rescale intensities (scale_intensity)

## Evidence

- [methods] Remove the precursor peak.: "Remove the precursor peak."
- [discussion] spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools: [evidence span withheld — anchor status: not_found]
- [readme] Common spectrum processing operations (precursor & noise peak removal, intensity filtering, intensity scaling): "Common spectrum processing operations (precursor & noise peak removal, intensity filtering, intensity scaling)"
