---
name: proforma-peptide-spectrum-annotation
description: Use when an MS/MS spectrum corresponds to a known peptide sequence and
  peaks must be labeled as fragment ions (e.g. a, b, y series, including modified
  residues such as oxidized methionine) for publication-quality figures.
when_to_use_negative:
- You need to read the spectral file (mzML/mgf) itself — spectrum_utils does not provide
  file-reading functionality and relies on external tools for that step.
- The peptide sequence is unknown, since annotation requires a ProForma peptide string
  to generate theoretical fragment m/z values.
- Interactive or static plotting is not needed and only raw peak lists are required
  downstream.
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: spectrum_utils
  role: Provides the annotate_proforma function that matches theoretical ProForma
    fragment ions to observed peaks and attaches annotations used by its plotting
    backends
  repo: https://github.com/bittremieux/spectrum_utils
- name: Python
  role: Runtime environment for spectrum_utils (Python 3.6+), in which the annotation
    call is made
- name: matplotlib
  role: Static plotting backend that renders the annotated spectrum into a publication-quality
    figure (e.g. savefig to PNG)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:053482958a4c4bb834225b4799e3047e5e18f955d88e78b83e6cc3d3e3a85cf3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Python
  - matplotlib
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# proforma-peptide-spectrum-annotation

## Summary

Annotate MS/MS spectrum peaks with peptide fragment assignments by passing a ProForma 2.0 peptide string to spectrum_utils's annotate_proforma, so that a-, b-, and y-ion series are color-labeled on the plot with a user-defined fragment tolerance.

## When to use

Use when an MS/MS spectrum corresponds to a known peptide sequence and peaks must be labeled as fragment ions (e.g. a, b, y series, including modified residues such as oxidized methionine) for publication-quality figures.

## When NOT to use

- You need to read the spectral file (mzML/mgf) itself — spectrum_utils does not provide file-reading functionality and relies on external tools for that step.
- The peptide sequence is unknown, since annotation requires a ProForma peptide string to generate theoretical fragment m/z values.
- Interactive or static plotting is not needed and only raw peak lists are required downstream.

## Inputs

- spectrum_utils MsmsSpectrum object with m/z and intensity peak arrays
- ProForma 2.0 peptide string with optional modification notation (e.g. DLTDYLM[Oxidation]K)
- fragment mass tolerance value (e.g. 0.5 Da)
- ion type selector string (e.g. 'aby')

## Outputs

- spectrum object with per-peak fragment-ion annotations attached
- annotated spectrum figure (static matplotlib or interactive altair plot) with color-labeled fragment ion peaks

## How to apply

Load the spectrum first as a spectrum_utils MsmsSpectrum object (e.g. from a USI via MsmsSpectrum.from_usi or from local peak data), because annotation operates on the processed in-memory spectrum, not a raw file. Call spectrum.annotate_proforma with a ProForma 2.0 string encoding the peptide and its modifications (example from the paper: DLTDYLM[Oxidation]K), supplying a fragment mass tolerance (0.5 Da in the paper's mirror-plot example) and the ion_types argument, set to 'aby' to label a, b, and y fragment series. The function matches each fragment ion's theoretical m/z against the observed peaks within the given tolerance and attaches fragment annotations to matching peaks. If peaks should already be denoised before annotation, precede it with set_mz_range (100–1400 m/z), remove_precursor_peak, and filter_intensity (keep peaks at at least 5% of base-peak intensity, limit to the 50 most intense peaks) so noise peaks do not receive spurious fragment labels. Annotated peaks are then rendered automatically by the static plotting (spectrum_utils.plot.spectrum or mirror) or the interactive (altair) backends, with unmatched peaks drawn unannotated.

## Related tools

- **spectrum_utils** (Provides the annotate_proforma function that matches theoretical ProForma fragment ions to observed peaks and attaches annotations used by its plotting backends) — https://github.com/bittremieux/spectrum_utils
- **Python** (Runtime environment for spectrum_utils (Python 3.6+), in which the annotation call is made)
- **matplotlib** (Static plotting backend that renders the annotated spectrum into a publication-quality figure (e.g. savefig to PNG))

## Examples

```
spectrum.annotate_proforma('DLTDYLM[Oxidation]K', fragment_tol=0.5, ion_types='aby')
```

## Evaluation signals

- Peaks within the fragment tolerance of theoretical a/b/y ion m/z values carry fragment annotations; non-matching peaks remain unannotated.
- The ProForma string parses correctly, including embedded modification notation such as M[Oxidation] in DLTDYLM[Oxidation]K.
- The rendered figure shows ion-series labels consistent with the requested ion_types ('aby') and the chosen fragment tolerance (0.5 Da in the paper's example).
- Annotated spectrum survives the switch between static and interactive backends, since 'it is trivial to switch between the static plotting functionality and the interactive plotting functionality'.
- With overly strict tolerance or wrong ion_types, most peaks appear unannotated — a visible symptom of mis-specified parameters.

## Limitations

- spectrum_utils does not provide functionality to read spectral data files; the spectrum must be loaded or obtained through external tools first.
- Annotation quality depends on the fragment tolerance; the article's worked example uses a relatively permissive 0.5 Da, so tighter tolerances may leave peaks unannotated.
- Only ion types supported by the ion_types argument (e.g. 'aby') are labeled; other fragment series must be explicitly requested.

## Evidence

- [methods] Annotate peaks corresponding to a, b, and y peptide fragments in the spectrum based on a ProForma 2.0 peptide string.: "Annotate peaks corresponding to a, b, and y peptide fragments in the spectrum based on a ProForma 2.0 peptide string."
- [other] Annotate both spectra with annotate_proforma for the peptide DLTDYLM[Oxidation]K using a fragment tolerance of 0.5 Da and ion_types='aby'.: [evidence span withheld — anchor status: not_found]
- [discussion] it is trivial to switch between the static plotting functionality to produce high-quality spectrum figures to include in scientific manuscripts and the interactive plotting functionality.: [evidence span withheld — anchor status: not_found]
- [discussion] spectrum_utils does not provide functionality to read spectral data files: "spectrum_utils does not provide functionality to read spectral data files"
- [methods] Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks.: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks."
