---
name: spectrum-intensity-filtering-and-noise-removal
description: Use when a raw MS/MS spectrum (e.g. loaded via USI from a public resource
  like GNPS) contains precursor peaks, off-range m/z values, and low-intensity noise
  peaks that must be cleaned before computational analysis or visualization.
when_to_use_negative:
- 'You need to read spectral data files from disk: spectrum_utils deliberately does
  not provide file-reading functionality and expects data provided by external tools
  (e.g. pymzML for mzML).'
- The spectrum is already processed (precursor removed, noise filtered, intensities
  scaled) — reapplying filter_intensity or scale_intensity would distort the peak
  list.
- You require end-to-end monolithic MS data processing covering quantification or
  identification rather than spectrum-level cleaning and visualization.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: spectrum_utils
  role: Provides the processing functions set_mz_range, remove_precursor_peak, filter_intensity,
    and scale_intensity applied to MsmsSpectrum objects
  repo: https://github.com/bittremieux/spectrum_utils
- name: Python
  role: Runtime language in which the processing chain is executed (spectrum_utils
    supports Python 3.6+)
- name: Pyteomics
  role: Companion proteomics library in the Python ecosystem used alongside spectrum_utils
    for related data-handling tasks
- name: pymzML
  role: External tool expected to supply spectra read from mzML files as input to
    spectrum_utils processing
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:38fd447c66eb64712e66b67a4719f1684471504565a90cb0c373c88c883c9950
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Pyteomics
  - Python
  - pymzML
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# spectrum-intensity-filtering-and-noise-removal

## Summary

A standard spectrum_utils preprocessing chain that cleans MS/MS spectra by fixing the m/z range, removing the precursor ion peak, filtering low-intensity noise peaks, and sqrt-scaling intensities so downstream analysis works on a reduced, denoised peak list.

## When to use

Use when a raw MS/MS spectrum (e.g. loaded via USI from a public resource like GNPS) contains precursor peaks, off-range m/z values, and low-intensity noise peaks that must be cleaned before computational analysis or visualization.

## When NOT to use

- You need to read spectral data files from disk: spectrum_utils deliberately does not provide file-reading functionality and expects data provided by external tools (e.g. pymzML for mzML).
- The spectrum is already processed (precursor removed, noise filtered, intensities scaled) — reapplying filter_intensity or scale_intensity would distort the peak list.
- You require end-to-end monolithic MS data processing covering quantification or identification rather than spectrum-level cleaning and visualization.

## Inputs

- A single MS/MS spectrum accessible via a Universal Spectrum Identifier (USI) from an online proteomics/metabolomics resource such as GNPS
- MS/MS spectra from MGF files (e.g. iPRG2012.mgf) loaded as a spectrum_utils MsmsSpectrum object

## Outputs

- A processed MsmsSpectrum object with precursor peak removed, m/z restricted to 100–1400, at most 50 peaks above 5% base-peak intensity, and sqrt-scaled intensities
- Serialized processed spectrum (e.g. processed_spectrum.pkl) for downstream steps

## How to apply

Load a single MS/MS spectrum with sus.MsmsSpectrum.from_usi using a Universal Spectrum Identifier, then apply four processing operations in order on the iPRG2012-style spectrum: restrict the m/z range with set_mz_range(min_mz=100, max_mz=1400) to filter out irrelevant peaks; remove the precursor peak with remove_precursor_peak(fragment_tol_mass=10, fragment_tol_mode='ppm'); filter intensity with filter_intensity(min_intensity=0.05, max_num_peaks=50), retaining only peaks at at least 5% of the base peak intensity and at most the 50 most intense peaks; and de-emphasize overly intense peaks with scale_intensity('root'), i.e. square-root scaling. The rationale is that precursor and noise peaks carry little fragment information and that sqrt scaling rebalances intensity dynamics before analysis or plotting. Serialize the resulting MsmsSpectrum object (e.g. pickle to processed_spectrum.pkl) when the processed spectrum feeds a later workflow step.

## Related tools

- **spectrum_utils** (Provides the processing functions set_mz_range, remove_precursor_peak, filter_intensity, and scale_intensity applied to MsmsSpectrum objects) — https://github.com/bittremieux/spectrum_utils
- **Python** (Runtime language in which the processing chain is executed (spectrum_utils supports Python 3.6+))
- **Pyteomics** (Companion proteomics library in the Python ecosystem used alongside spectrum_utils for related data-handling tasks)
- **pymzML** (External tool expected to supply spectra read from mzML files as input to spectrum_utils processing) — https://github.com/pymzml/pymzML

## Examples

```
spectrum = sus.MsmsSpectrum.from_usi(usi); spectrum.set_mz_range(min_mz=100, max_mz=1400); spectrum.remove_precursor_peak(10, 'ppm'); spectrum.filter_intensity(0.05, 50); spectrum.scale_intensity('root')
```

## Evaluation signals

- The processed MsmsSpectrum object loads successfully after pickling/unpickling (processed_spectrum.pkl).
- All peaks in the output peak list fall within the 100–1400 m/z range set by set_mz_range.
- No peak in the output matches the precursor m/z within the 10 ppm fragment tolerance used by remove_precursor_peak.
- Peak count is at most max_num_peaks=50 and every retained peak is at least 5% of the base peak intensity.
- Peak intensities are monotonic w.r.t. the originals after square-root scaling (relative ordering preserved, dynamic range compressed).

## Limitations

- spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task; input spectra must come from external tools or USI loading.
- The document is a bioRxiv preprint that was not certified by peer review at the time of the snapshot.
- Peaks are removed by hard thresholds (5% base peak, top 50 peaks, 100–1400 m/z); legitimately informative low-intensity or out-of-range fragments are discarded along with noise.

## Evidence

- [methods] Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks.: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks."
- [methods] Scale the peak intensities by their square root to de-emphasize overly intense peaks.: "Scale the peak intensities by their square root to de-emphasize overly intense peaks."
- [methods] Restrict the mass range to 100–1400 m/z to filter out irrelevant peaks.: [evidence span withheld — anchor status: not_found]
- [methods] Spectrum processing in spectrum_utils consists of fixing the m/z range, removing precursor ion peaks and low-intensity noise peaks, and scaling the peak intensities by their square root.: [evidence span withheld — anchor status: not_found]
- [readme] Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism.: "Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism."
