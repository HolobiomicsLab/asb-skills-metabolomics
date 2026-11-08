---
name: spectrum-processing-chain-assembly
description: Use when raw MS/MS spectra (e.g. from an iPRG2012 MGF file or a USI-resolvable
  online resource) need to be cleaned and normalized before downstream analysis such
  as deep neural network processing, peptide fragmentation annotation, or spectrum
  plotting.
when_to_use_negative:
- spectrum_utils cannot read spectral data files itself — do not use this skill as
  the file-parsing step; first load data with an external reader such as Pyteomics
  or pymzML and pass the spectra in
- Do not apply when a monolithic end-to-end MS processing tool already covers the
  experimental use case and no customized processing chain is needed
- Not intended for quantification workflows — the skill covers spectrum cleaning/scaling,
  not MSnbase-style quantification in R
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: spectrum_utils
  role: 'Provides the MsmsSpectrum processing API: set_mz_range, remove_precursor_peak,
    filter_intensity, scale_intensity, annotate_proforma'
  repo: https://github.com/bittremieux/spectrum_utils
- name: Pyteomics
  role: External reader used to load MS/MS spectra (e.g. from iPRG2012.mgf) since
    spectrum_utils does not parse spectral data files
- name: pymzML
  role: Alternative external tool to efficiently read and process spectra in the mzML
    format before applying the processing chain
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:c79c91d66df10e7b556ac5d96019e38fc8337e7bf99ad7f87605a7ecfce35e65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Pyteomics
  - Python
  - pymzML
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# spectrum-processing-chain-assembly

## Summary

Assembles the standard spectrum_utils MS/MS processing chain — m/z range restriction, precursor peak removal, noise peak filtering, and intensity scaling — on a spectrum loaded via a Universal Spectrum Identifier (USI), following the iPRG2012 example workflow.

## When to use

Use when raw MS/MS spectra (e.g. from an iPRG2012 MGF file or a USI-resolvable online resource) need to be cleaned and normalized before downstream analysis such as deep neural network processing, peptide fragmentation annotation, or spectrum plotting.

## When NOT to use

- spectrum_utils cannot read spectral data files itself — do not use this skill as the file-parsing step; first load data with an external reader such as Pyteomics or pymzML and pass the spectra in
- Do not apply when a monolithic end-to-end MS processing tool already covers the experimental use case and no customized processing chain is needed
- Not intended for quantification workflows — the skill covers spectrum cleaning/scaling, not MSnbase-style quantification in R

## Inputs

- A single MS/MS spectrum referenced by a Universal Spectrum Identifier (USI) from an online proteomics/metabolomics resource
- MS/MS spectra read from standard file formats such as iPRG2012.mgf (via external tools like Pyteomics or pymzML)
- Optional: ProForma 2.0 peptide string for peak annotation

## Outputs

- A processed MsmsSpectrum object with restricted m/z range (100–1400), precursor peak removed, ≤50 peaks above 5% base peak intensity, and square-root-scaled intensities
- A serialized pickle of the processed spectrum (e.g. processed_spectrum.pkl)
- Optionally, an annotated spectrum with a, b, and y fragment peak assignments

## How to apply

Load a single MS/MS spectrum with sus.MsmsSpectrum.from_usi (or from the iPRG2012.mgf file via Pyteomics). First restrict the m/z range with set_mz_range(min_mz=100, max_mz=1400) to discard irrelevant peaks outside the analytically useful window. Remove the precursor ion peak with remove_precursor_peak(fragment_tol_mass=10, fragment_tol_mode='ppm') so unfragmented precursor signal does not dominate. Filter noise with filter_intensity(min_intensity=0.05, max_num_peaks=50), retaining only peaks at ≥5% of the base peak intensity and at most the 50 most intense peaks. Finally scale intensities with scale_intensity('root') so square-root-transformed intensities de-emphasize overly intense peaks; optionally annotate a/b/y fragment peaks from a ProForma 2.0 peptide string with annotate_proforma before serializing the MsmsSpectrum (e.g. processed_spectrum.pkl).

## Related tools

- **spectrum_utils** (Provides the MsmsSpectrum processing API: set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity, annotate_proforma) — https://github.com/bittremieux/spectrum_utils
- **Pyteomics** (External reader used to load MS/MS spectra (e.g. from iPRG2012.mgf) since spectrum_utils does not parse spectral data files)
- **pymzML** (Alternative external tool to efficiently read and process spectra in the mzML format before applying the processing chain) — https://github.com/pymzml/pymzML

## Examples

```
sus.MsmsSpectrum.from_usi(usi); spec.set_mz_range(100, 1400); spec.remove_precursor_peak(10, 'ppm'); spec.filter_intensity(0.05, 50); spec.scale_intensity('root')
```

## Evaluation signals

- The serialized processed_spectrum.pkl loads successfully as an MsmsSpectrum object
- All retained peaks fall within the 100–1400 m/z window set by set_mz_range
- No peak lies within 10 ppm of the precursor m/z (precursor peak removed)
- Peak list contains at most 50 peaks, all at ≥5% of the base peak intensity
- Peak intensities are square-root transformed (root scaling), de-emphasizing overly intense peaks

## Limitations

- spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task — loading must be delegated to external tools
- The article is a bioRxiv preprint that was not certified by peer review
- No changelog was found for the repository snapshot, complicating version pinning of the processing API

## Evidence

- [methods] Processing steps: [evidence span withheld — anchor status: not_found]
- [methods] Noise filtering: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks."
- [methods] Intensity scaling: "Scale the peak intensities by their square root to de-emphasize overly intense peaks."
- [readme] USI loading: "Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism."
- [discussion] File reading limitation: [evidence span withheld — anchor status: not_found]
