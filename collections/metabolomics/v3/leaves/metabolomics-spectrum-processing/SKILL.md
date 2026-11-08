---
name: metabolomics-spectrum-processing
description: Use when an MS/MS spectrum (e.g., a GNPS spectral-library entry referenced
  by USI) must be processed — noise/precursor peak removal, intensity filtering and
  scaling — and rendered as a static annotated spectrum figure.
when_to_use_negative:
- 'Reading raw spectral data files: spectrum_utils does not provide functionality
  to read spectral data files and relies on external tools (e.g., pymzML) for this
  task'
- Interactive web-based spectrum plots when a static publication figure is required
  is a mismatch in the other direction — but for purely interactive exploratory analysis
  the static matplotlib path is not the intended output; use the interactive plotting
  functionality instead
- Proteomics peptide-centric workflows needing ProForma 2.0 annotation of a, b, and
  y fragments are supported by annotate_proforma but differ from the metabolomics
  SMILES-based annotation path
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: spectrum_utils
  role: Core package for MS/MS spectrum loading via USI, processing (set_mz_range,
    remove_precursor_peak, filter_intensity, scale_intensity), SMILES-based annotation,
    and static plotting
  repo: https://github.com/bittremieux/spectrum_utils
- name: Python
  role: Runtime language; spectrum_utils is available for Python 3.6+
- name: Matplotlib
  role: Backend for rendering the static publication-quality spectrum figure via spectrum_utils.plot.spectrum
- name: pymzML
  role: External tool for reading mzML-format spectra, which spectrum_utils does not
    provide itself
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:e944cc7a881574897b487105edaad6d8e5f6dc62b652a32d5984efc31a0f210c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Matplotlib
  - Python
  - pymzML
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabolomics-spectrum-processing

## Summary

Process and visualize metabolomics MS/MS spectra with the spectrum_utils Python package: load spectra by Universal Spectrum Identifier, apply the standard filtering chain (m/z range restriction, precursor peak removal, intensity filtering, root intensity scaling), and render publication-quality static spectrum figures via matplotlib.

## When to use

Use when an MS/MS spectrum (e.g., a GNPS spectral-library entry referenced by USI) must be processed — noise/precursor peak removal, intensity filtering and scaling — and rendered as a static annotated spectrum figure.

## When NOT to use

- Reading raw spectral data files: spectrum_utils does not provide functionality to read spectral data files and relies on external tools (e.g., pymzML) for this task
- Interactive web-based spectrum plots when a static publication figure is required is a mismatch in the other direction — but for purely interactive exploratory analysis the static matplotlib path is not the intended output; use the interactive plotting functionality instead
- Proteomics peptide-centric workflows needing ProForma 2.0 annotation of a, b, and y fragments are supported by annotate_proforma but differ from the metabolomics SMILES-based annotation path

## Inputs

- MS/MS spectrum referenced by a Universal Spectrum Identifier (e.g., mzspec:GNPSLIBRARY:CCMSLIB00000840351 for a GNPS spectral-library entry)
- SMILES string encoding a small-molecule (sub)structure for fragment peak annotation
- Fragment mass tolerance parameters (fragment_tol_mass, fragment_tol_mode) for precursor peak removal

## Outputs

- Processed MsmsSpectrum object with restricted m/z range (100–1400), precursor peak removed, at most 50 most-intense peaks above 5% base peak intensity, and root-scaled intensities
- Publication-quality static spectrum figure (PNG) with colored annotated fragment peaks rendered via matplotlib

## How to apply

Load the spectrum over the network with spectrum_utils.spectrum.MsmsSpectrum.from_usi (e.g., mzspec:GNPSLIBRARY:CCMSLIB00000840351 fetches the GNPS spectral-library entry). Chain the processing calls in order: set_mz_range(min_mz=100, max_mz=1400) to restrict the mass range, remove_precursor_peak(fragment_tol_mass, fragment_tol_mode) to remove the precursor peak, filter_intensity(min_intensity=0.05, max_num_peaks=50) to drop low-intensity noise peaks below 5% of the base peak and keep only the 50 most intense peaks, and scale_intensity('root') to scale intensities by their square root and de-emphasize overly intense peaks. Fragment peaks can then be annotated — e.g., from a SMILES string for a small-molecule (sub)structure via the appropriate MsmsSpectrum annotation method. Render the annotated spectrum with spectrum_utils.plot.spectrum (grid disabled, top/right axes despined) using matplotlib, and save the figure as a PNG.

## Related tools

- **spectrum_utils** (Core package for MS/MS spectrum loading via USI, processing (set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity), SMILES-based annotation, and static plotting) — https://github.com/bittremieux/spectrum_utils
- **Python** (Runtime language; spectrum_utils is available for Python 3.6+)
- **Matplotlib** (Backend for rendering the static publication-quality spectrum figure via spectrum_utils.plot.spectrum)
- **pymzML** (External tool for reading mzML-format spectra, which spectrum_utils does not provide itself) — https://github.com/pymzml/pymzML

## Examples

```
from spectrum_utils import spectrum, plot; spec = spectrum.MsmsSpectrum.from_usi('mzspec:GNPSLIBRARY:CCMSLIB00000840351').set_mz_range(100, 1400).remove_precursor_peak(0.5, 'Da').filter_intensity(0.05, 50).scale_intensity('root'); plot.spectrum(spec)
```

## Evaluation signals

- The output PNG file exists and is loadable as an image
- The rendered figure visually contains the processed spectrum with colored annotated fragment peaks
- Processing invariants: m/z restricted to 100–1400, precursor peak removed, no more than 50 peaks retained, all retained peaks at ≥ 5% of the base peak intensity
- Intensities are root-scaled (square-root transformation applied), so overly intense peaks are de-emphasized
- The USI load succeeds over the network for the GNPS library entry (mzspec:GNPSLIBRARY:CCMSLIB00000840351)

## Limitations

- spectrum_utils does not provide functionality to read spectral data files; it relies on external tools for this task
- from_usi requires network access to fetch the GNPS spectral-library entry
- The article describing the workflow was a bioRxiv preprint that was not certified by peer review

## Evidence

- [methods] The m/z range is set to 100–1400 m/z.: [evidence span withheld — anchor status: not_found]
- [methods] Low-intensity noise peaks are removed.: "Low-intensity noise peaks are removed."
- [methods] Scale the peak intensities by their square root to de-emphasize overly intense peaks.: "Scale the peak intensities by their square root to de-emphasize overly intense peaks."
- [abstract] spectrum_utils is a Python package for mass spectrometry data processing and visualization: "spectrum_utils is a Python package for mass spectrometry data processing and visualization"
- [readme] Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism.: "Spectrum loading from online proteomics and metabolomics data resources using the [Universal Spectrum Identifier (USI)](https://www.psidev.info/usi) mechanism."
- [discussion] spectrum_utils does not provide functionality to read spectral data files: "spectrum_utils does not provide functionality to read spectral data files"
