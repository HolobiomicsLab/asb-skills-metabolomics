---
name: mirror-spectrum-plot-visualization
description: Use when two MS/MS spectra need to be compared visually (e.g. a library
  spectrum vs. an unknown query spectrum from GNPS), to assess peak-by-peak similarity,
  including annotated peptide fragment ions (a/b/y).
when_to_use_negative:
- You only need to inspect a single spectrum — use the regular static or interactive
  spectrum plot rather than a mirror plot
- You need to read raw spectral data files (e.g. mzML) directly — spectrum_utils does
  not provide file reading; use pymzML or another external tool first and supply data
  as MsmsSpectrum objects
- The spectra have not been noise-filtered and low-intensity peaks would overwhelm
  the fragment-ion comparison
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: spectrum_utils
  role: Provides MsmsSpectrum.from_usi to load spectra from USIs, annotate_proforma
    for fragment annotation, and plot.mirror for rendering the two-spectrum comparison
  repo: https://github.com/bittremieux/spectrum_utils
- name: matplotlib
  role: Figure backend for the static mirror plot; used to create the 12x6 figure
    and save mirror.png via savefig
- name: Python
  role: Runtime environment (Python 3.6+ per the article) in which the spectrum_utils
    plotting workflow is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:93ac7d63c9919b7f4282b73566ed744fff9128be19a1a61f815121f08cfd9359
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Python
  - matplotlib
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mirror-spectrum-plot-visualization

## Summary

Render a mirror plot with spectrum_utils to compare two MS/MS spectra: the top spectrum is plotted normally and the comparison spectrum is inverted below it, making matched (and unmatched) peaks between the two spectra visually apparent. This is the standard way to show spectral similarity, e.g. library-versus-query spectrum matches.

## When to use

Use when two MS/MS spectra need to be compared visually (e.g. a library spectrum vs. an unknown query spectrum from GNPS), to assess peak-by-peak similarity, including annotated peptide fragment ions (a/b/y).

## When NOT to use

- You only need to inspect a single spectrum — use the regular static or interactive spectrum plot rather than a mirror plot
- You need to read raw spectral data files (e.g. mzML) directly — spectrum_utils does not provide file reading; use pymzML or another external tool first and supply data as MsmsSpectrum objects
- The spectra have not been noise-filtered and low-intensity peaks would overwhelm the fragment-ion comparison

## Inputs

- Universal Spectrum Identifier (USI) string for the top spectrum (e.g. mzspec:MSV000079960:DY_HS_Exp7-Ad1:scan:30372)
- Universal Spectrum Identifier (USI) string for the bottom/comparison spectrum (e.g. mzspec:MSV000080679:j11962_C1orf144:scan:10671)
- ProForma 2.0 peptide string for annotation (e.g. DLTDYLM[Oxidation]K)
- Fragment ion tolerance and ion types (0.5 Da, ion_types='aby')

## Outputs

- A mirror plot figure with the top spectrum upright and the bottom spectrum inverted, showing shared annotated fragment peaks
- PNG image (mirror.png) saved at dpi=300 with bbox_inches='tight' and transparent background
- Optionally an interactive spectrum plot for web pages, trivially switchable from the static plotting backend

## How to apply

Load both spectra as MsmsSpectrum objects using MsmsSpectrum.from_usi with Universal Spectrum Identifiers (e.g. mzspec:MSV000079960:DY_HS_Exp7-Ad1:scan:30372 for the top spectrum and mzspec:MSV000080679:j11962_C1orf144:scan:10671 for the bottom spectrum), retrieving them from online proteomics repositories. Annotate both spectra with annotate_proforma for the peptide DLTDYLM[Oxidation]K using a fragment tolerance of 0.5 Da and ion_types='aby' so matching fragment peaks are colored consistently across the two spectra. Render the comparison with spectrum_utils.plot.mirror(spectrum_top, spectrum_bottom) into a 12x6 matplotlib figure; the bottom spectrum is automatically plotted inverted. Optionally preprocess both spectra first (set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity) so noise peaks do not clutter the comparison, then save the figure via matplotlib's savefig with dpi=300, bbox_inches='tight', and a transparent background.

## Related tools

- **spectrum_utils** (Provides MsmsSpectrum.from_usi to load spectra from USIs, annotate_proforma for fragment annotation, and plot.mirror for rendering the two-spectrum comparison) — https://github.com/bittremieux/spectrum_utils
- **matplotlib** (Figure backend for the static mirror plot; used to create the 12x6 figure and save mirror.png via savefig)
- **Python** (Runtime environment (Python 3.6+ per the article) in which the spectrum_utils plotting workflow is executed)

## Examples

```
import spectrum_utils.plot as sup, spectrum_utils.spectrum as sus; top=sus.MsmsSpectrum.from_usi('mzspec:MSV000079960:DY_HS_Exp7-Ad1:scan:30372'); bot=sus.MsmsSpectrum.from_usi('mzspec:MSV000080679:j11962_C1orf144:scan:10671'); sup.mirror(top, bot)
```

## Evaluation signals

- The figure contains two stacked axes: the top spectrum plotted upright and the bottom spectrum plotted inverted (negative intensity direction), which is the defining feature of a mirror plot
- Annotated fragment peaks (a/b/y ions of DLTDYLM[Oxidation]K) are colored consistently across both spectra; matched peaks appear at the same m/z in the top and bottom panels
- The saved mirror.png is 300 dpi with tight bounding box and transparent background
- Both spectra load successfully from their USIs (mzspec:MSV000079960:DY_HS_Exp7-Ad1:scan:30372 and mzspec:MSV000080679:j11962_C1orf144:scan:10671) without retrieval errors
- Preprocessing (set_mz_range 100–1400 m/z, remove_precursor_peak, filter_intensity to >=5% base peak and top 50 peaks) visibly reduces noise peaks in the rendered comparison

## Limitations

- spectrum_utils does not provide functionality to read spectral data files; spectra must come from USI retrieval or be supplied by external tools such as pymzML
- Fragment annotation requires a valid ProForma 2.0 peptide string and a suitable fragment tolerance; the workflow uses 0.5 Da, which may be inappropriate for high-resolution data
- Static plotting requires matplotlib; the interactive (web) plotting path uses a different backend and is meant for explorative analysis or web resources

## Evidence

- [other] spectrum_utils includes a mirror plot function that plots one spectrum on top and a comparison spectrum at the bottom (inverted), clearly showing similarity between the two spectra: [evidence span withheld — anchor status: not_found]
- [methods] annotate_proforma finding: "Annotate peaks corresponding to a, b, and y peptide fragments in the spectrum based on a ProForma 2.0 peptide string."
- [methods] filter steps: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks."
- [discussion] static vs interactive switching: [evidence span withheld — anchor status: not_found]
- [readme] USI loading from README: "Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism."
- [discussion] no file reading: [evidence span withheld — anchor status: not_found]
