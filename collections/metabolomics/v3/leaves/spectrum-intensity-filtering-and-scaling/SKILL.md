---
name: spectrum-intensity-filtering-and-scaling
description: Use when raw MS/MS spectra (e.g. from an MGF file like iPRG2012.mgf)
  contain precursor peaks and low-intensity noise that must be removed and intensities
  normalized before spectral library searching, comparison, or plotting.
when_to_use_negative:
- When the input is a spectral data file that still needs to be read — spectrum_utils
  does not provide file-reading functionality and relies on external tools such as
  Pyteomics or pymzML for that task
- When spectra are already fully processed (precursor removed, noise-filtered, intensities
  scaled) — reapplying the chain would further alter peak intensities and counts
- When custom end-to-end MS data processing is required that monolithic tools cannot
  cover — this skill only covers the standard peak-processing operations, not custom
  analysis
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: spectrum_utils
  role: 'Primary processing library: provides set_mz_range, remove_precursor_peak,
    filter_intensity, and scale_intensity operations optimized for computational efficiency'
  repo: https://github.com/bittremieux/spectrum_utils
- name: Pyteomics
  role: Reads spectra from the iPRG2012.mgf input file since spectrum_utils does not
    read spectral data files itself
- name: pymzML
  role: Alternative processor used to replicate the same filtering and scaling steps
    on mzML data for runtime comparison
  repo: https://github.com/pymzml/pymzML
- name: pyOpenMS
  role: Alternative processor using MascotGenericFile loading plus ThresholdMower,
    NLargest, and SqrtMower filters with equivalent parameters
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:e3f26640f38f453ff89c0da8f6b01945b0f3b3ef8ad1e547940763f2b87eafb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Pyteomics
  - pyOpenMS
  - pymzML
  - seaborn
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spectrum-intensity-filtering-and-scaling

## Summary

Applies spectrum_utils' optimized peak-processing chain to MS/MS spectra — m/z range restriction, precursor peak removal, intensity filtering, and square-root intensity scaling — to remove noise and de-emphasize overly intense peaks before search or visualization.

## When to use

Use when raw MS/MS spectra (e.g. from an MGF file like iPRG2012.mgf) contain precursor peaks and low-intensity noise that must be removed and intensities normalized before spectral library searching, comparison, or plotting.

## When NOT to use

- When the input is a spectral data file that still needs to be read — spectrum_utils does not provide file-reading functionality and relies on external tools such as Pyteomics or pymzML for that task
- When spectra are already fully processed (precursor removed, noise-filtered, intensities scaled) — reapplying the chain would further alter peak intensities and counts
- When custom end-to-end MS data processing is required that monolithic tools cannot cover — this skill only covers the standard peak-processing operations, not custom analysis

## Inputs

- MS/MS peak lists (e.g. MGF file such as iPRG2012.mgf) read via Pyteomics or pymzML
- Individual MsmsSpectrum objects with peak m/z and intensity arrays

## Outputs

- Filtered, root-scaled MsmsSpectrum objects with peaks restricted to 100–1400 m/z, no precursor peak, and low-intensity noise peaks removed
- Per-spectrum processing runtimes for benchmarking plots (e.g. seaborn boxplot saved as runtime.png at 300 dpi)

## How to apply

Load MS/MS spectra externally (spectrum_utils does not read files itself — use Pyteomics for MGF or pymzML for mzML, discarding spectra with fewer than 10 peaks or missing charge information). Then apply the spectrum_utils processing chain in order: set_mz_range(100, 1400) to restrict the mass range; remove_precursor_peak(0.02 Da) to remove the precursor peak; filter_intensity with min_intensity=0.05 (peaks at ≥5% of base peak intensity) and a max_num_peaks cap to drop low-intensity noise; and scale_intensity('root') to scale intensities by their square root so overly intense peaks are de-emphasized. The rationale is that spectrum_utils' processing is optimized for computational efficiency, making it suitable for large spectral volumes and for benchmarking against pymzML (reduce to 100–1400 m/z, remove precursor, remove noise, sqrt-scale) or pyOpenMS (ThresholdMower, NLargest, SqrtMower) equivalents.

## Related tools

- **spectrum_utils** (Primary processing library: provides set_mz_range, remove_precursor_peak, filter_intensity, and scale_intensity operations optimized for computational efficiency) — https://github.com/bittremieux/spectrum_utils
- **Pyteomics** (Reads spectra from the iPRG2012.mgf input file since spectrum_utils does not read spectral data files itself)
- **pymzML** (Alternative processor used to replicate the same filtering and scaling steps on mzML data for runtime comparison) — https://github.com/pymzml/pymzML
- **pyOpenMS** (Alternative processor using MascotGenericFile loading plus ThresholdMower, NLargest, and SqrtMower filters with equivalent parameters)

## Examples

```
from spectrum_utils import spectrum as sus; s = sus.MsmsSpectrum('spec', precursor_mz, 2, mz, intensity); s.set_mz_range(100, 1400).remove_precursor_peak(0.02).filter_intensity(0.05, 150).scale_intensity('root')
```

## Evaluation signals

- All retained peaks fall within the 100–1400 m/z window after set_mz_range
- No peak remains at the precursor m/z after remove_precursor_peak with the 0.02 Da tolerance
- Retained peaks are all at ≥5% of the base peak intensity and their count respects the configured max_num_peaks cap (workflow uses 150; article methods text cites restricting to the most intense peaks)
- After scale_intensity('root'), intensities are square-root transformed — the most intense peak no longer dominates the intensity range
- For benchmarking use, the seaborn boxplot (runtime.png, 300 dpi) shows spectrum_utils processing times lower than pymzML and pyOpenMS

## Limitations

- spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task — an external reader is mandatory
- The exact max_num_peaks setting differs between the article methods text (50 most intense peaks) and the reproducible workflow (max_num_peaks=150); choose and document one explicitly
- Monolithic end-to-end MS data processing software cannot cover all possible experimental use cases; this chain covers only standard noise-removal and scaling operations

## Evidence

- [abstract] the data processing functionality is highly optimized for computational eﬀiciency to be able to deal with the large volumes of data: "the data processing functionality is highly optimized for computational eﬀiciency to be able to deal with the large volumes of data"
- [methods] Restrict the mass range to 100–1400 m/z to filter out irrelevant peaks.: [evidence span withheld — anchor status: not_found]
- [methods] Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks"
- [methods] Scale the peak intensities by their square root to de-emphasize overly intense peaks.: "Scale the peak intensities by their square root to de-emphasize overly intense peaks."
- [discussion] spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task.: [evidence span withheld — anchor status: not_found]
- [readme] Common spectrum processing operations (precursor & noise peak removal, intensity filtering, intensity scaling): "Common spectrum processing operations (precursor & noise peak removal, intensity filtering, intensity scaling)"
