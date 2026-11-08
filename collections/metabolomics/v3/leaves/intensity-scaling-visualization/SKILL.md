---
name: intensity-scaling-visualization
description: Use when MS/MS spectra (e.g. from an MGF file such as iPRG2012.mgf) need
  intensity de-emphasis of overly intense peaks and downstream rendering as static
  publication figures or interactive web plots.
when_to_use_negative:
- You need to read mzML/MGF files directly — spectrum_utils does not provide file-reading
  functionality and relies on external tools such as pymzML or pyteomics for this
- End-to-end turnkey MS data processing is required; monolithic end-to-end software
  cannot cover all use cases, whereas spectrum_utils is a focused processing/visualization
  layer
- Non-Python environments (e.g. R, where MSnbase covers similar tasks)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: spectrum_utils
  role: 'Primary package: performs set_mz_range, remove_precursor_peak, filter_intensity,
    scale_intensity, annotate_proforma, and static/interactive spectrum plotting'
  repo: https://github.com/bittremieux/spectrum_utils
- name: Python
  role: Runtime environment; spectrum_utils is available for Python 3.6+
- name: pymzML
  role: Alternative/comparison library for reading and processing mzML spectra (e.g.
    reduce, remove_precursor_peak, remove_noise, normalize)
  repo: https://github.com/pymzml/pymzML
- name: Pyteomics
  role: External reader used to load spectra from iPRG2012.mgf for processing by spectrum_utils
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:7c703d70615f45411054e326d72254fb09400a76dbec868fef9d27240d62aab4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Pyteomics
  - Python
  - pyOpenMS
  - pymzML
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# intensity-scaling-visualization

## Summary

Peak-intensity scaling (e.g. square-root transformation) and spectrum visualization as implemented by the spectrum_utils Python package, which optimizes these operations for computational efficiency on large MS/MS volumes and renders publication-quality or interactive figures.

## When to use

Use when MS/MS spectra (e.g. from an MGF file such as iPRG2012.mgf) need intensity de-emphasis of overly intense peaks and downstream rendering as static publication figures or interactive web plots.

## When NOT to use

- You need to read mzML/MGF files directly — spectrum_utils does not provide file-reading functionality and relies on external tools such as pymzML or pyteomics for this
- End-to-end turnkey MS data processing is required; monolithic end-to-end software cannot cover all use cases, whereas spectrum_utils is a focused processing/visualization layer
- Non-Python environments (e.g. R, where MSnbase covers similar tasks)

## Inputs

- MS/MS spectra loaded from a standard format (e.g. iPRG2012.mgf read via pyteomics.mgf) — spectrum_utils itself requires data supplied by an external reader
- Optional ProForma 2.0 peptide string for fragment peak annotation

## Outputs

- Processed spectrum objects with square-root-scaled intensities (scale_intensity('root', 1)), filtered to 100–1400 m/z and the top 150 peaks ≥5% base peak
- Publication-quality static spectrum figures or interactive spectrum plots for web pages (e.g. runtime.png boxplots in the paper's benchmark)

## How to apply

Load spectra with an external reader (spectrum_utils does not read spectral files itself, so use e.g. pyteomics.mgf on iPRG2012.mgf), then apply the processing chain: set_mz_range(100, 1400) to restrict to the 100–1400 m/z range, remove_precursor_peak(0.02, 'Da'), filter_intensity(0.05, 150) to keep peaks at ≥5% of the base peak up to the most intense 150 peaks, and scale_intensity('root', 1) to scale intensities by their square root. Scale peaks via annotate_proforma when a ProForma 2.0 peptide string is available (a, b, y fragments). Rationale: square-root scaling de-emphasizes overly intense peaks, and spectrum_utils is optimized for computational efficiency on large spectral volumes — note the first call incurs a Numba JIT compilation cost, a one-time outlier. Finally, produce either static publication-quality figures or interactive (Vega-embed-based) plots; switching between the two modes is trivial.

## Related tools

- **spectrum_utils** (Primary package: performs set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity, annotate_proforma, and static/interactive spectrum plotting) — https://github.com/bittremieux/spectrum_utils
- **Python** (Runtime environment; spectrum_utils is available for Python 3.6+)
- **pymzML** (Alternative/comparison library for reading and processing mzML spectra (e.g. reduce, remove_precursor_peak, remove_noise, normalize)) — https://github.com/pymzml/pymzML
- **Pyteomics** (External reader used to load spectra from iPRG2012.mgf for processing by spectrum_utils)

## Examples

```
import spectrum_utils.spectrum as sus; spec = sus.MsmsSpectrum('id', precursor_mz, charge, mz, intensity); spec = spec.set_mz_range(100, 1400).remove_precursor_peak(0.02, 'Da').filter_intensity(0.05, 150).scale_intensity('root', 1)
```

## Evaluation signals

- The processed spectrum retains peaks only within 100–1400 m/z and at ≥5% of the base peak intensity, capped at the 150 most intense peaks
- Peak intensities are square-root transformed (scale_intensity('root', 1)), so overly intense peaks are de-emphasized
- Per-spectrum processing time after the first call is much lower than pymzML 2.4.4 or pyOpenMS 2.4.0 equivalents (ParentPeakMower, ThresholdMower 0.05, NLargest 150, SqrtMower); the first spectrum_utils call is a known Numba JIT outlier
- Generated figure is publication-quality (e.g. runtime.png, log-scaled boxplot) or an interactive Vega-embed plot, with trivial switching between the two modes

## Limitations

- spectrum_utils does not provide functionality to read spectral data files and relies on external tools for this task
- The first processing call triggers Numba JIT compilation, which appears as a significant runtime outlier
- The article's preprint was not certified by peer review, and no changelog was found for the repository snapshot

## Evidence

- [methods] Scale the peak intensities by their square root to de-emphasize overly intense peaks.: "Scale the peak intensities by their square root to de-emphasize overly intense peaks."
- [methods] Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks.: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks."
- [discussion] spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task.: [evidence span withheld — anchor status: not_found]
- [abstract] The visualization functionality makes it possible to easily produce publication-quality figures as well as interactive spectrum plots for inclusion on web pages.: "The visualization functionality makes it possible to easily produce publication-quality figures as well as interactive spectrum plots for inclusion on web pages."
- [readme] spectrum_utils is a Python package for efficient mass spectrometry data processing and visualization.: "spectrum_utils is a Python package for efficient mass spectrometry data processing and visualization."
