---
name: mass-spectrometry-spectrum-preprocessing
description: Use when raw MS/MS spectra (e.g. from an .mgf file like iPRG2012.mgf)
  must be cleaned and standardized before downstream analysis such as library searching
  or deep-neural-network processing, and peak lists need mz-range restriction, precursor/noise
  removal, and intensity scaling.
when_to_use_negative:
- When you need to read spectral data files directly — spectrum_utils does not provide
  file-reading functionality and relies on external tools such as Pyteomics or pymzML
  for this task.
- When a monolithic end-to-end MS data processing tool already covers the exact experimental
  setup, since spectrum_utils is designed for customized prototyping workflows instead.
- When publication-quality or interactive spectrum visualization is the only goal,
  with no processing step required.
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: spectrum_utils
  role: 'Core processing library: set_mz_range, remove_precursor_peak, filter_intensity,
    scale_intensity operations with computational-efficiency focus'
  repo: https://github.com/bittremieux/spectrum_utils
- name: Pyteomics
  role: Reads spectra from iPRG2012.mgf and applies the <10 peaks / missing-charge
    discard filters before preprocessing
- name: pymzML
  role: 'Baseline processing comparison: replicate mz-range reduction, precursor removal,
    noise removal, and sqrt scaling with per-spectrum timing'
  repo: https://github.com/pymzml/pymzML
- name: pyOpenMS
  role: 'Alternative baseline: MascotGenericFile loading plus ThresholdMower, NLargest,
    and SqrtMower filters with equivalent parameters'
- name: seaborn
  role: Log-scale boxplot of collected processing runtimes across the three toolkits
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:7c20ffe9d11e57c0b9820fd07e95b1ee7e68a1249692f54d5a92f22f1434db59
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
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mass-spectrometry-spectrum-preprocessing

## Summary

Efficient MS/MS spectrum preprocessing with spectrum_utils — restricting the m/z range, removing precursor peaks, filtering low-intensity noise peaks, and scaling intensities — as described in Bittremieux (2019, Analytical Chemistry). The package's processing is optimized for computational efficiency on large spectral volumes, demonstrated on the iPRG2012.mgf dataset against pymzML and pyOpenMS baselines.

## When to use

Use when raw MS/MS spectra (e.g. from an .mgf file like iPRG2012.mgf) must be cleaned and standardized before downstream analysis such as library searching or deep-neural-network processing, and peak lists need mz-range restriction, precursor/noise removal, and intensity scaling.

## When NOT to use

- When you need to read spectral data files directly — spectrum_utils does not provide file-reading functionality and relies on external tools such as Pyteomics or pymzML for this task.
- When a monolithic end-to-end MS data processing tool already covers the exact experimental setup, since spectrum_utils is designed for customized prototyping workflows instead.
- When publication-quality or interactive spectrum visualization is the only goal, with no processing step required.

## Inputs

- MGF file of MS/MS spectra (e.g. iPRG2012.mgf)
- Spectra with at least 10 peaks and valid charge annotation

## Outputs

- Preprocessed spectra: restricted to 100–1400 m/z, precursor peak removed, intensity-filtered to top peaks ≥5% of base peak, sqrt-scaled intensities
- Per-spectrum processing runtimes for spectrum_utils, pymzML, and pyOpenMS
- Boxplot figure (runtime.png, 300 dpi) comparing processing times on a log y-axis

## How to apply

Load spectra from an MGF file with an external reader (e.g. Pyteomics), discarding spectra with fewer than 10 peaks or missing charge information, since spectrum_utils itself does not read spectral data files. Then apply the spectrum_utils processing chain: set_mz_range(100, 1400) to restrict the mass range, remove_precursor_peak(0.02 Da) to remove the precursor peak, filter_intensity(min_intensity=0.05, max_num_peaks=150) to retain only peaks at at least 5% of the base peak intensity up to a peak cap, and scale_intensity('root') to square-root-scale intensities and de-emphasize overly intense peaks. Rationale: spectrum_utils is optimized for computational efficiency, so this chain should outperform equivalent pymzML or pyOpenMS (ThresholdMower, NLargest, SqrtMower) filter chains when timed per spectrum; visualize the timing comparison with a seaborn boxplot on a log y-axis and save as runtime.png at 300 dpi.

## Related tools

- **spectrum_utils** (Core processing library: set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity operations with computational-efficiency focus) — https://github.com/bittremieux/spectrum_utils
- **Pyteomics** (Reads spectra from iPRG2012.mgf and applies the <10 peaks / missing-charge discard filters before preprocessing)
- **pymzML** (Baseline processing comparison: replicate mz-range reduction, precursor removal, noise removal, and sqrt scaling with per-spectrum timing) — https://github.com/pymzml/pymzML
- **pyOpenMS** (Alternative baseline: MascotGenericFile loading plus ThresholdMower, NLargest, and SqrtMower filters with equivalent parameters)
- **seaborn** (Log-scale boxplot of collected processing runtimes across the three toolkits)

## Examples

```
mgf_filename = "iPRG2012.mgf"; spectrum.set_mz_range(100, 1400); spectrum.remove_precursor_peak(0.02); spectrum.filter_intensity(0.05, 150); spectrum.scale_intensity('root')
```

## Evaluation signals

- runtime.png exists at 300 dpi and its log-scale boxplot shows spectrum_utils processing times lower than pymzML and pyOpenMS, consistent with the reported computational efficiency advantage
- Each output spectrum has m/z restricted to the 100–1400 range and no residual precursor peak (0.02 Da removal tolerance applied)
- Retained peaks are at least 5% of base peak intensity and peak count respects the filtering cap (150 in the timing workflow, 50 in the paper's example)
- Intensities are square-root scaled (de-emphasizing overly intense peaks) in all processed spectra
- Spectra with fewer than 10 peaks or missing charge information were excluded before preprocessing

## Limitations

- spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task — an external reader is required.
- The article describes customized prototyping workflows; monolithic end-to-end MS software may already cover some experimental use cases.
- The source article is a bioRxiv preprint that was not certified by peer review.

## Evidence

- [readme] spectrum_utils is a Python package for efficient mass spectrometry data processing and visualization.: "spectrum_utils is a Python package for efficient mass spectrometry data processing and visualization."
- [methods] filter_intensity removes low-intensity noise peaks, retaining peaks at at least 5% of base peak intensity: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks."
- [methods] scale_intensity scales intensities by square root to de-emphasize overly intense peaks: "Scale the peak intensities by their square root to de-emphasize overly intense peaks."
- [discussion] spectrum_utils does not read spectral data files, relying on external tools: [evidence span withheld — anchor status: not_found]
- [intro] Computational efficiency is a key feature for processing large amounts of spectral data: [evidence span withheld — anchor status: not_found]
