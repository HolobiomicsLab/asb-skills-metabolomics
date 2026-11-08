---
name: msms-spectrum-noise-filtering
description: Use when an MS/MS spectrum (e.g. loaded from a USI such as mzspec:PXD004732:...:scan:41840)
  contains precursor peaks, noise peaks, or overly intense peaks that must be removed
  or de-emphasized before fragment annotation or publication-quality plotting.
when_to_use_negative:
- When the input is raw vendor or mzML data that still needs to be read and converted
  — spectrum_utils does not provide functionality to read spectral data files because
  excellent external tools (e.g. pymzML) already perform this task.
- 'When the goal is spectral identification or peptide-spectrum matching rather than
  spectrum cleanup: spectrum_utils takes MS data provided by external tools as input
  for subsequent processing and visualization, it does not perform identification.'
- When the spectrum is already clean (no precursor peak, no low-intensity noise peaks)
  — applying filter_intensity with a 5% base-peak threshold would unnecessarily discard
  real fragment peaks.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: spectrum_utils
  role: Provides the spectrum processing chain (set_mz_range, remove_precursor_peak,
    filter_intensity, scale_intensity, annotate_proforma) applied to MS/MS spectra
    identified by USI
  repo: https://github.com/bittremieux/spectrum_utils
- name: pymzML
  role: External tool to efficiently read and process spectra in the mzML format and
    provide the input spectrum to spectrum_utils for subsequent processing and visualization
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:23c85d73b5defd5de3bfbe71cbdf0440b7e16ed4abf307c5fac489f76e3aa023
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Python
  - pymzML
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# msms-spectrum-noise-filtering

## Summary

Filter and clean MS/MS spectra prior to visualization or downstream analysis using spectrum_utils: restrict the m/z range, remove the precursor peak, drop low-intensity noise peaks, and rescale intensities so that peak annotation and plotting operate on a clean, de-noised spectrum.

## When to use

Use when an MS/MS spectrum (e.g. loaded from a USI such as mzspec:PXD004732:...:scan:41840) contains precursor peaks, noise peaks, or overly intense peaks that must be removed or de-emphasized before fragment annotation or publication-quality plotting.

## When NOT to use

- When the input is raw vendor or mzML data that still needs to be read and converted — spectrum_utils does not provide functionality to read spectral data files because excellent external tools (e.g. pymzML) already perform this task.
- When the goal is spectral identification or peptide-spectrum matching rather than spectrum cleanup: spectrum_utils takes MS data provided by external tools as input for subsequent processing and visualization, it does not perform identification.
- When the spectrum is already clean (no precursor peak, no low-intensity noise peaks) — applying filter_intensity with a 5% base-peak threshold would unnecessarily discard real fragment peaks.

## Inputs

- An MS/MS spectrum referenced by a Universal Spectrum Identifier (USI), e.g. mzspec:PXD004732:01650b_BC2-TUM_first_pool_53_01_01-3xHCD-1h-R2:scan:41840
- An MsmsSpectrum object from spectrum_utils.spectrum
- Optional: a mass-spectral data file (e.g. MGF) loaded by an external tool, since spectrum_utils itself does not read spectral data files

## Outputs

- A processed MsmsSpectrum with the m/z range restricted to 100–1400 m/z, the precursor peak removed (10 ppm tolerance), noise peaks filtered (≥5% of base peak intensity, ≤50 most intense peaks), and root-scaled intensities
- A cleaned peak list suitable for fragment annotation via annotate_proforma and for plotting via spectrum_utils.plot.spectrum

## How to apply

Load the spectrum via its Universal Spectrum Identifier with spectrum_utils.spectrum.MsmsSpectrum.from_usi, then apply a chain of processing steps in order. First restrict the mass range with set_mz_range(min_mz=100, max_mz=1400) to filter out irrelevant peaks. Second remove the precursor peak with remove_precursor_peak(fragment_tol=10, unit='ppm') so that the precursor itself is not annotated as a fragment. Third filter intensities with filter_intensity(min_intensity=0.05, max_num_peaks=50), retaining only peaks at ≥5% of the base peak intensity and at most the 50 most intense peaks — this is the noise-removal step that drops low-intensity noise peaks. Fourth scale intensities with scale_intensity('root') (square-root scaling) to de-emphasize overly intense peaks. The rationale, grounded in the spectrum_utils paper, is that low-intensity noise peaks would otherwise clutter annotation and visualization, and the precursor peak would confound fragment assignment; the 100–1400 m/z window, 5% base-peak threshold, 50-peak cap, 10 ppm precursor tolerance, and root intensity scaling are the paper's concrete defaults for a clean, publication-ready spectrum.

## Related tools

- **spectrum_utils** (Provides the spectrum processing chain (set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity, annotate_proforma) applied to MS/MS spectra identified by USI) — https://github.com/bittremieux/spectrum_utils
- **pymzML** (External tool to efficiently read and process spectra in the mzML format and provide the input spectrum to spectrum_utils for subsequent processing and visualization) — https://github.com/pymzml/pymzML

## Examples

```
from spectrum_utils.spectrum import MsmsSpectrum; s = MsmsSpectrum.from_usi('mzspec:PXD004732:01650b_BC2-TUM_first_pool_53_01_01-3xHCD-1h-R2:scan:41840'); s.set_mz_range(min_mz=100, max_mz=1400); s.remove_precursor_peak(10, 'ppm'); s.filter_intensity(0.05, 50); s.scale_intensity('root')
```

## Evaluation signals

- Confirm the processed spectrum no longer contains the precursor peak: re-run remove_precursor_peak with fragment_tol=10 ppm and verify no peaks are removed in the second pass.
- Verify that every remaining peak has intensity ≥ 0.05 × base-peak intensity and that the total number of peaks is ≤ 50, matching the filter_intensity parameters.
- Check that the m/z range of all retained peaks falls within 100–1400 m/z after set_mz_range.
- Confirm intensity scaling is applied by inspecting that the new intensities are square roots of the original (scale_intensity('root')).
- After annotation with annotate_proforma('WNQLQAFWGTGK', 10, 'ppm', ion_types='aby'), verify that fragment peaks carry b/y/a annotations and the final 300-dpi PNG (quickstart.png) renders the annotated spectrum per the paper's Figure 1A workflow.

## Limitations

- spectrum_utils does not provide functionality to read spectral data files — the spectrum must be loaded from an external source (e.g. via USI or a tool like pymzML) before processing.
- The filtering parameters (100–1400 m/z window, 5% base-peak threshold, 50-peak cap, 10 ppm precursor tolerance, root intensity scaling) are the paper's defaults and may need tuning for other instrument types or acquisition methods.
- The document is a bioRxiv preprint that was not certified by peer review at the time of this snapshot; no changelog was found for the repository snapshot.

## Evidence

- [methods] Restrict the mass range to 100–1400 m/z to filter out irrelevant peaks.: [evidence span withheld — anchor status: not_found]
- [methods] Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks.: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks."
- [methods] Scale the peak intensities by their square root to de-emphasize overly intense peaks.: "Scale the peak intensities by their square root to de-emphasize overly intense peaks."
- [discussion] spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task.: [evidence span withheld — anchor status: not_found]
- [readme] Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism.: "Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism."
