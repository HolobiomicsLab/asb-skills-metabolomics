---
name: static-spectrum-plot-rendering
description: Use when the goal is a static, publication-quality annotated MS/MS spectrum
  figure from a USI-addressed proteomics or metabolomics spectrum, e.g. replicating
  Figure 1A of the spectrum_utils paper.
when_to_use_negative:
- You need to read local spectral data files (mzML/MGF) directly — spectrum_utils
  does not provide file-reading functionality and relies on external tools such as
  pymzML for that task
- The target output is an interactive web-embedded spectrum plot rather than a static
  publication figure, though the paper notes it is trivial to switch between the two
  plotting modes
- The input is already a processed/derived artifact (e.g. feature table) rather than
  an MS/MS spectrum addressable by a USI
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: spectrum_utils
  role: USI spectrum loading, processing (set_mz_range, remove_precursor_peak, filter_intensity,
    scale_intensity), ProForma/SMILES annotation, and static spectrum plotting
  repo: https://github.com/bittremieux/spectrum_utils
- name: matplotlib
  role: Figure backend (figsize 12x6 axis) on which spectrum_utils.plot.spectrum renders
    the static figure saved at 300 dpi
- name: pymzML
  role: Example external tool for reading mzML-format spectra that can feed data into
    spectrum_utils, since spectrum_utils itself does not read spectral files
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:2b31e21682300ef2aa421f3d93004cbc3ec201c4b88f6f50aba9a13c166a49cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Matplotlib
  - Python
  - matplotlib
  - pymzML
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# static-spectrum-plot-rendering

## Summary

Render a processed MS/MS spectrum as a publication-quality static figure using spectrum_utils.plot.spectrum on a matplotlib axis, after USI-based loading, peak filtering, and fragment annotation (ProForma peptide for proteomics or SMILES for metabolomics).

## When to use

Use when the goal is a static, publication-quality annotated MS/MS spectrum figure from a USI-addressed proteomics or metabolomics spectrum, e.g. replicating Figure 1A of the spectrum_utils paper.

## When NOT to use

- You need to read local spectral data files (mzML/MGF) directly — spectrum_utils does not provide file-reading functionality and relies on external tools such as pymzML for that task
- The target output is an interactive web-embedded spectrum plot rather than a static publication figure, though the paper notes it is trivial to switch between the two plotting modes
- The input is already a processed/derived artifact (e.g. feature table) rather than an MS/MS spectrum addressable by a USI

## Inputs

- Universal Spectrum Identifier (USI) string, e.g. mzspec:PXD004732:01650b_BC2-TUM_first_pool_53_01_01-3xHCD-1h-R2:scan:41840 or mzspec:GNPSLIBRARY:CCMSLIB00000840351
- Peptide sequence as a ProForma 2.0 string (proteomics annotation)
- SMILES string for the annotated (sub)structure (metabolomics annotation)

## Outputs

- Static PNG figure (e.g. quickstart.png) at 300 dpi showing the processed spectrum with colored annotated fragment peaks
- In-memory matplotlib figure/axis objects with grid disabled and right/top spines removed

## How to apply

Load the spectrum by Universal Spectrum Identifier with MsmsSpectrum.from_usi (e.g. mzspec:PXD004732:...:scan:41840 from ProteomeXchange, or mzspec:GNPSLIBRARY:CCMSLIB00000840351 from the GNPS library, fetched over the network). Process it with set_mz_range(min_mz=100, max_mz=1400), remove_precursor_peak (10 ppm fragment tolerance), filter_intensity(min_intensity=0.05, max_num_peaks=50), and scale_intensity('root') to de-emphasize overly intense peaks. Annotate fragments via annotate_proforma('WNQLQAFWGTGK', 10, 'ppm', ion_types='aby') for peptides, or the SMILES-based annotation path for small molecules. Plot on a matplotlib figure (figsize 12x6) with spectrum_utils.plot.spectrum(grid=False, ax=ax), remove the right/top spines, and save as PNG at 300 dpi.

## Related tools

- **spectrum_utils** (USI spectrum loading, processing (set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity), ProForma/SMILES annotation, and static spectrum plotting) — https://github.com/bittremieux/spectrum_utils
- **matplotlib** (Figure backend (figsize 12x6 axis) on which spectrum_utils.plot.spectrum renders the static figure saved at 300 dpi)
- **pymzML** (Example external tool for reading mzML-format spectra that can feed data into spectrum_utils, since spectrum_utils itself does not read spectral files) — https://github.com/pymzml/pymzML

## Examples

```
from spectrum_utils.spectrum import MsmsSpectrum; spec = MsmsSpectrum.from_usi('mzspec:GNPSLIBRARY:CCMSLIB00000840351'); spec.set_mz_range(min_mz=100, max_mz=1400); spec.filter_intensity(min_intensity=0.05, max_num_peaks=50)
```

## Evaluation signals

- Output PNG (e.g. quickstart.png) exists, is loadable as an image, and is 300 dpi
- Spectrum m/z axis is restricted to 100–1400 m/z and the precursor peak is removed within 10 ppm fragment tolerance
- Peak count is capped at the 50 most intense peaks, each ≥5% of base peak intensity, with root-scaled intensities
- Annotated fragment peaks are highlighted in color per the ion-type scheme (a, b, y for proteomics; structure-derived annotations for metabolomics)
- Figure styling matches the paper: grid=False and right/top spines removed

## Limitations

- spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task; it takes MS data provided by external tools as input
- from_usi fetches spectra over the network (e.g. GNPS spectral-library entries), so offline environments will fail
- The source document is a bioRxiv preprint that was not certified by peer review, and no changelog was found for the repository snapshot

## Evidence

- [abstract] The visualization functionality makes it possible to easily produce publication-quality figures as well as interactive spectrum plots for inclusion on web pages.: "The visualization functionality makes it possible to easily produce publication-quality figures as well as interactive spectrum plots for inclusion on web pages."
- [methods] Restrict the mass range to 100–1400 m/z to filter out irrelevant peaks.: [evidence span withheld — anchor status: not_found]
- [methods] Annotate peaks corresponding to a, b, and y peptide fragments in the spectrum based on a ProForma 2.0 peptide string.: "Annotate peaks corresponding to a, b, and y peptide fragments in the spectrum based on a ProForma 2.0 peptide string."
- [discussion] It is trivial to switch between the static plotting functionality to produce high-quality spectrum figures to include in scientific manuscripts and the interactive plotting functionality.: [evidence span withheld — anchor status: not_found]
- [readme] spectrum_utils is a Python package for efficient mass spectrometry data processing and visualization.: "spectrum_utils is a Python package for efficient mass spectrometry data processing and visualization."
