---
name: square-root-intensity-scaling
description: Use when an MS/MS spectrum's peak intensities are dominated by a few
  very intense peaks (e.g. after precursor and noise-peak removal) and downstream
  analysis or visualization needs a compressed dynamic range.
when_to_use_negative:
- spectrum_utils does not provide functionality to read spectral data files — use
  pymzML or another reader first; this skill is not for raw file import.
- The spectrum's peaks have not yet had the precursor peak removed and noise filtered
  — scaling the unfiltered peak list would still leave the base peak and noise dominating
  the transformed intensities.
- A downstream tool requires linear (untransformed) intensities, e.g. absolute quantification-style
  comparisons rather than spectral pattern matching or visualization.
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: spectrum_utils
  role: Provides the scale_intensity('root') call (and the preceding set_mz_range,
    remove_precursor_peak, filter_intensity processing) on MsmsSpectrum objects
  repo: https://github.com/bittremieux/spectrum_utils
- name: Python
  role: Runtime environment; spectrum_utils is available for Python 3.6+
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:acc9622301c77b749d614eb9332cd7bb5db8a17d759f2f367f882d01b4def058
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Pyteomics
  - Python
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# square-root-intensity-scaling

## Summary

Scale MS/MS peak intensities by their square root using spectrum_utils' scale_intensity('root') to de-emphasize overly intense base peaks while preserving relative peak ordering.

## When to use

Use when an MS/MS spectrum's peak intensities are dominated by a few very intense peaks (e.g. after precursor and noise-peak removal) and downstream analysis or visualization needs a compressed dynamic range.

## When NOT to use

- spectrum_utils does not provide functionality to read spectral data files — use pymzML or another reader first; this skill is not for raw file import.
- The spectrum's peaks have not yet had the precursor peak removed and noise filtered — scaling the unfiltered peak list would still leave the base peak and noise dominating the transformed intensities.
- A downstream tool requires linear (untransformed) intensities, e.g. absolute quantification-style comparisons rather than spectral pattern matching or visualization.

## Inputs

- MsmsSpectrum object with raw or partially processed peak intensities (e.g. loaded from a USI via sus.MsmsSpectrum.from_usi or from iPRG2012.mgf)

## Outputs

- MsmsSpectrum with square-root-scaled peak intensities (same peak m/z positions, transformed intensity values)

## How to apply

After basic spectrum conditioning — restricting the m/z range (e.g. set_mz_range(min_mz=100, max_mz=1400)), removing the precursor peak (remove_precursor_peak with fragment_tol_mass=10, fragment_tol_mode='ppm'), and filtering low-intensity noise (filter_intensity(min_intensity=0.05, max_num_peaks=50) retaining peaks at >=5% of base peak intensity and at most 50 peaks) — apply scale_intensity('root') to the MsmsSpectrum. The square-root transform compresses the intensity dynamic range so overly intense peaks no longer dominate, which is the rationale given in the paper's processing example (an iPRG2012 spectrum loaded via sus.MsmsSpectrum.from_usi). Apply it as the last processing step before visualization or serialization (e.g. pickling to processed_spectrum.pkl), since it depends on the cleaned peak list rather than the raw one.

## Related tools

- **spectrum_utils** (Provides the scale_intensity('root') call (and the preceding set_mz_range, remove_precursor_peak, filter_intensity processing) on MsmsSpectrum objects) — https://github.com/bittremieux/spectrum_utils
- **Python** (Runtime environment; spectrum_utils is available for Python 3.6+)

## Examples

```
sus.MsmsSpectrum.from_usi(usi).set_mz_range(min_mz=100, max_mz=1400).remove_precursor_peak(fragment_tol_mass=10, fragment_tol_mode='ppm').filter_intensity(min_intensity=0.05, max_num_peaks=50).scale_intensity('root')
```

## Evaluation signals

- Re-load the pickled output (processed_spectrum.pkl) and confirm the peak count is unchanged (no peaks added/removed) relative to the pre-scaling spectrum.
- Verify each transformed intensity equals the square root of its pre-scaling value within floating-point tolerance, and that the m/z array is identical.
- Confirm the ordering of peaks by intensity is preserved (root scaling is monotonic).
- Confirm the pipeline order: scaling applied after filter_intensity(min_intensity=0.05, max_num_peaks=50), so at most 50 peaks remain and all are >=5% of the (pre-scaling) base peak intensity.
- On an interactive or static plot, the base peak no longer visually dwarfs mid-intensity fragment peaks as it does in the unscaled spectrum.

## Limitations

- The paper describes square-root scaling as de-emphasizing overly intense peaks — it is a heuristic for visualization/pattern analysis, not a physical intensity correction.
- spectrum_utils relies on external tools to read spectral data files, so the spectrum must be loaded via a USI or another reader before scaling can be applied.
- The referenced document is a bioRxiv preprint that was not certified by peer review at that snapshot.

## Evidence

- [methods] Scale the peak intensities by their square root to de-emphasize overly intense peaks.: "Scale the peak intensities by their square root to de-emphasize overly intense peaks."
- [methods] Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks.: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks."
- [readme] Common spectrum processing operations (precursor & noise peak removal, intensity filtering, intensity scaling): "Common spectrum processing operations (precursor & noise peak removal, intensity filtering, intensity scaling)"
- [discussion] spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task.: [evidence span withheld — anchor status: not_found]
