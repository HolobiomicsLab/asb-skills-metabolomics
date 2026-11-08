---
name: proforma-peptidoform-parsing
description: Use when an MS/MS spectrum must be peak-annotated against a known peptide
  sequence supplied as a ProForma 2.0 string (optionally with PTMs), e.g. to build
  a fragment-annotation table for a USI-resolved spectrum.
when_to_use_negative:
- spectrum_utils provides no functionality to read spectral data files itself — for
  pure mzML/MGF ingestion without peak annotation, use an external reader such as
  pymzML directly
- The peptide sequence is unknown or only a de novo guess; annotation requires a defined
  peptidoform to generate theoretical fragments
- The goal is generic spectral processing (m/z restriction, precursor removal, intensity
  filtering/scaling) without fragment assignment
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: spectrum_utils
  role: Provides MsmsSpectrum.from_usi spectrum loading and annotate_proforma for
    ProForma 2.0-based fragment peak annotation at a user-specified ppm tolerance
    and ion types
  repo: https://github.com/bittremieux/spectrum_utils
- name: pymzML
  role: External reader for mzML spectral data that can feed spectra into annotation
    workflows, since spectrum_utils itself does not read spectral files
  repo: pymzml/pymzML
- name: Pyteomics
  role: Proteomics data processing companion in Python for peptide/fragment handling
    around the annotation workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:75cc7f915a7b9b9279821f3488417b5700d897bb08559ad6b59544d76bf4c2ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Pyteomics
  - Python
  - pymzML
  - spectrum_utils
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# proforma-peptidoform-parsing

## Summary

Annotate MS/MS spectrum peaks with theoretical peptide fragment ions by parsing a ProForma 2.0 peptidoform string in spectrum_utils, supporting fragment types such as a/b/y ions, PTMs at specified positions, charge, and isotope annotations.

## When to use

Use when an MS/MS spectrum must be peak-annotated against a known peptide sequence supplied as a ProForma 2.0 string (optionally with PTMs), e.g. to build a fragment-annotation table for a USI-resolved spectrum.

## When NOT to use

- spectrum_utils provides no functionality to read spectral data files itself — for pure mzML/MGF ingestion without peak annotation, use an external reader such as pymzML directly
- The peptide sequence is unknown or only a de novo guess; annotation requires a defined peptidoform to generate theoretical fragments
- The goal is generic spectral processing (m/z restriction, precursor removal, intensity filtering/scaling) without fragment assignment

## Inputs

- Universal Spectrum Identifier (USI) string resolving an MS/MS spectrum from an online proteomics resource (e.g. mzspec:PXD004732:...:scan:41840)
- ProForma 2.0 peptidoform string (e.g. WNQLQAFWGTGK), optionally with position-specific modifications
- Fragment mass tolerance (e.g. 10 ppm) and ion type selection (e.g. 'aby')
- Optionally an mzML/MGF spectrum file read via external tools (e.g. pymzML, iPRG2012.mgf)

## Outputs

- Fragment annotation table mapping matched peak m/z values to fragment ion type, charge, and isotope annotation (e.g. fragment_annotations.csv)
- Annotated MsmsSpectrum object suitable for plotting (static publication figures or interactive Vega plots)

## How to apply

Load the target MS/MS spectrum by Universal Spectrum Identifier via MsmsSpectrum.from_usi (e.g. mzspec:PXD004732:01650b_BC2-TUM_first_pool_53_01_01-3xHCD-1h-R2:scan:41840). Parse the ProForma 2.0 peptidoform (e.g. WNQLQAFWGTGK) to obtain the amino-acid sequence and any modifications at specified positions, from which theoretical fragment ion series are derived. Preprocess the spectrum as in the paper's workflow — restrict m/z (e.g. 100–1400), remove the precursor peak, filter low-intensity noise peaks, and optionally scale intensities — so that annotation is not confounded by noise. Call annotate_proforma with a fragment mass tolerance (10 ppm in the exemplar workflow) and ion_types='aby' to match observed peaks to a, b, and y fragments; by default b/y ions are used. Collect matched peak m/z values together with their fragment ion type, charge, and isotope annotations into a table (e.g. fragment_annotations.csv). Validate that every assigned fragment corresponds to an a, b, or y match of the peptidoform within the stated tolerance.

## Related tools

- **spectrum_utils** (Provides MsmsSpectrum.from_usi spectrum loading and annotate_proforma for ProForma 2.0-based fragment peak annotation at a user-specified ppm tolerance and ion types) — https://github.com/bittremieux/spectrum_utils
- **pymzML** (External reader for mzML spectral data that can feed spectra into annotation workflows, since spectrum_utils itself does not read spectral files) — pymzml/pymzML
- **Pyteomics** (Proteomics data processing companion in Python for peptide/fragment handling around the annotation workflow)

## Examples

```
from spectrum_utils.spectrum import MsmsSpectrum; s = MsmsSpectrum.from_usi('mzspec:PXD004732:01650b_BC2-TUM_first_pool_53_01_01-3xHCD-1h-R2:scan:41840'); s.annotate_proforma('WNQLQAFWGTGK', 'aby', 10e-6)
```

## Evaluation signals

- Every fragment in the output table is an a, b, or y ion match of the given peptidoform (e.g. WNQLQAFWGTGK), not an arbitrary peak label
- All reported fragment m/z assignments fall within the specified tolerance (10 ppm in the exemplar workflow)
- Each annotation row carries fragment ion type, charge state, and isotope information, not just m/z
- Spectrum preprocessing preceded annotation: m/z range restriction (100–1400), precursor peak removal, and low-intensity noise filtering (≥5% base-peak intensity, top 50 peaks) per the paper's methods
- Unmatched peaks remain unannotated rather than being force-assigned, consistent with tolerance-based matching

## Limitations

- spectrum_utils does not provide functionality to read spectral data files and relies on external tools for that task
- Annotation quality depends on the supplied fragment tolerance; too-wide tolerance risks incorrect fragment assignments
- The source paper is a bioRxiv preprint that was not certified by peer review, and no changelog is available for the repository snapshot

## Evidence

- [methods] Annotate peaks corresponding to a, b, and y peptide fragments in the spectrum based on a ProForma 2.0 peptide string.: "Annotate peaks corresponding to a, b, and y peptide fragments in the spectrum based on a ProForma 2.0 peptide string."
- [methods] Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks.: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks"
- [readme] Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism.: "Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI)"
- [discussion] spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task.: [evidence span withheld — anchor status: not_found]
