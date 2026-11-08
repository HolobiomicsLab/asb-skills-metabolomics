---
name: mass-spectrometry-intensity-filtering
description: Use when MS/MS spectra contain many low-intensity noise peaks — e.g.
  prior to peak annotation, library comparison, or spectrum visualization — and you
  need to restrict peaks to a fraction of the base peak intensity and cap their number.
when_to_use_negative:
- spectrum_utils does not provide functionality to read spectral data files; do not
  use this skill as the file-reading step — obtain spectra from an external reader
  (e.g. pymzML) first.
- Spectra that are already denoised peak lists (few, high-quality fragment peaks)
  — further aggressive filtering (50-peak cap, 5% base-peak cutoff) could discard
  informative low-abundance fragments.
- Situations where low-intensity peaks carry the signal of interest (e.g. diagnostic
  trace metabolite fragments) — the 5%-of-base-peak threshold is tuned for noise removal,
  not trace-peak detection.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: spectrum_utils
  role: Provides the chainable MsmsSpectrum processing calls filter_intensity (min_intensity,
    max_num_peaks), scale_intensity ('root'), set_mz_range, and remove_precursor_peak
    used to implement the filtering workflow.
  repo: https://github.com/bittremieux/spectrum_utils
- name: pymzML
  role: External reader for mzML-format spectra that can supply input for subsequent
    processing and visualization in spectrum_utils.
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:8224a6ca376768b25ad6a55e243e691f2e18099d561fc5163c42c241505ea032
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Matplotlib
  - Python
  - pymzML
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# mass-spectrometry-intensity-filtering

## Summary

Intensity filtering for mass spectrometry spectra: retain only the most intense peaks and scale intensities to suppress low-intensity noise peaks before annotation or downstream analysis, using spectrum_utils' filter_intensity and scale_intensity operations.

## When to use

Use when MS/MS spectra contain many low-intensity noise peaks — e.g. prior to peak annotation, library comparison, or spectrum visualization — and you need to restrict peaks to a fraction of the base peak intensity and cap their number.

## When NOT to use

- spectrum_utils does not provide functionality to read spectral data files; do not use this skill as the file-reading step — obtain spectra from an external reader (e.g. pymzML) first.
- Spectra that are already denoised peak lists (few, high-quality fragment peaks) — further aggressive filtering (50-peak cap, 5% base-peak cutoff) could discard informative low-abundance fragments.
- Situations where low-intensity peaks carry the signal of interest (e.g. diagnostic trace metabolite fragments) — the 5%-of-base-peak threshold is tuned for noise removal, not trace-peak detection.

## Inputs

- MS/MS spectrum loaded via a Universal Spectrum Identifier (USI, e.g. mzspec:GNPSLIBRARY:CCMSLIB00000840351)
- Mass spectral data in standard file formats such as .mgf (e.g. iPRG2012.mgf) or mzML
- A spectrum_utils MsmsSpectrum object

## Outputs

- Filtered MsmsSpectrum with peaks restricted to ≥5% of the base peak intensity and at most the 50 most intense peaks
- Root-scaled peak intensities within the filtered spectrum

## How to apply

Load spectra (e.g. from a USI such as mzspec:GNPSLIBRARY:... or an MGF file like iPRG2012.mgf) and apply chainable processing calls on a spectrum_utils MsmsSpectrum object. First, optionally restrict the mass range with set_mz_range (e.g. 100–1400 m/z) and remove the precursor peak so that filtering targets fragment peaks only. Then call filter_intensity(min_intensity=0.05, max_num_peaks=50): peaks below 5% of the base peak intensity are discarded and the total number of peaks is capped at the 50 most intense, which removes low-intensity noise peaks. Finally, scale_intensity('root') (square-root scaling) de-emphasizes overly intense peaks so dominant peaks do not visually or numerically overshadow fragment peaks. Rationale: processing spectra into a consistent, noise-reduced peak set makes downstream annotation, comparison, and publication-quality plotting more informative.

## Related tools

- **spectrum_utils** (Provides the chainable MsmsSpectrum processing calls filter_intensity (min_intensity, max_num_peaks), scale_intensity ('root'), set_mz_range, and remove_precursor_peak used to implement the filtering workflow.) — https://github.com/bittremieux/spectrum_utils
- **pymzML** (External reader for mzML-format spectra that can supply input for subsequent processing and visualization in spectrum_utils.) — https://github.com/pymzml/pymzML

## Examples

```
from spectrum_utils.spectrum import MsmsSpectrum; spec = MsmsSpectrum.from_usi('mzspec:GNPSLIBRARY:CCMSLIB00000840351'); spec.filter_intensity(min_intensity=0.05, max_num_peaks=50); spec.scale_intensity('root')
```

## Evaluation signals

- After filter_intensity(min_intensity=0.05, max_num_peaks=50), no retained peak has intensity below 5% of the base peak intensity.
- The retained peak count is ≤ 50.
- After scale_intensity('root'), peak intensities are square-root scaled, and overly intense peaks are visually de-emphasized relative to their raw intensities.
- No precursor peak remains among the retained peaks if remove_precursor_peak was applied before filtering.
- Downstream annotation/plotting operates only on the filtered peak list — e.g. the rendered spectrum shows colored annotated fragment peaks on the reduced peak set.

## Limitations

- spectrum_utils does not itself read spectral data files; it takes MS data provided by external tools (e.g. pymzML for mzML) as input for subsequent processing.
- spectrum_utils emphasizes computational efficiency to handle large volumes of spectral data, but processing is generic — thresholds like the 5% base-peak cutoff and 50-peak cap are workflow-specific choices, not universal defaults.
- The intensity scaling step (e.g. square-root) de-emphasizes overly intense peaks, which is desirable for visualization but alters absolute intensity relationships for downstream quantitative comparisons.
- spectrum_utils takes the MS data provided by external tools as input, so filtering quality depends on the fidelity of the upstream peak picking / deconvolution.

## Evidence

- [methods] Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks.: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks."
- [methods] Scale the peak intensities by their square root to de-emphasize overly intense peaks.: "Scale the peak intensities by their square root to de-emphasize overly intense peaks."
- [discussion] spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task.: [evidence span withheld — anchor status: not_found]
- [discussion] Instead, spectrum_utils takes the MS data provided by such tools as input for subsequent processing and visualization.: [evidence span withheld — anchor status: not_found]
- [readme] Common spectrum processing operations (precursor & noise peak removal, intensity filtering, intensity scaling): "Common spectrum processing operations (precursor & noise peak removal, intensity filtering, intensity scaling)"
