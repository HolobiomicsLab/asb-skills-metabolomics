---
name: spectral-peak-annotation-tabulation
description: Use when an MS/MS spectrum (e.g. identified via a Universal Spectrum
  Identifier such as mzspec:PXD004732:...:scan:41840) must have its peaks matched
  to peptide fragment ions of a known peptidoform, and the matched annotations need
  to be exported as a structured table (e.
when_to_use_negative:
- The task is reading or parsing raw spectral data files — spectrum_utils does not
  provide file-reading functionality and relies on external tools (e.g. pymzML, Pyteomics)
  for that step.
- The peptide sequence or peptidoform is unknown, so no theoretical fragment series
  can be generated for peak matching.
- The goal is de novo spectrum interpretation or spectral library matching rather
  than matching peaks to a specified peptidoform's fragment ions.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: spectrum_utils
  role: Provides MsmsSpectrum.from_usi for loading the spectrum by USI and annotate_proforma
    for matching observed peaks to a/b/y fragment ions from the ProForma 2.0 peptidoform
    within a 10 ppm tolerance
  repo: https://github.com/bittremieux/spectrum_utils
- name: Pyteomics
  role: Proteomics data processing companion cited alongside spectrum_utils for proteomics
    tasks in Python
- name: Python
  role: Execution environment for the annotation and tabulation workflow (spectrum_utils
    requires Python 3.6+)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:c2e93a1f6b884ce4027da9561f2f86be1592a464f70e3ce9b37999be515ab460
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Pyteomics
  - Python
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spectral-peak-annotation-tabulation

## Summary

Annotate observed MS/MS spectrum peaks against theoretical peptide fragment ions (a, b, y) derived from a ProForma 2.0 peptidoform using spectrum_utils, then tabulate matched m/z values with their fragment type, charge, and isotope assignments.

## When to use

Use when an MS/MS spectrum (e.g. identified via a Universal Spectrum Identifier such as mzspec:PXD004732:...:scan:41840) must have its peaks matched to peptide fragment ions of a known peptidoform, and the matched annotations need to be exported as a structured table (e.g. CSV) for downstream validation or reporting.

## When NOT to use

- The task is reading or parsing raw spectral data files — spectrum_utils does not provide file-reading functionality and relies on external tools (e.g. pymzML, Pyteomics) for that step.
- The peptide sequence or peptidoform is unknown, so no theoretical fragment series can be generated for peak matching.
- The goal is de novo spectrum interpretation or spectral library matching rather than matching peaks to a specified peptidoform's fragment ions.

## Inputs

- Universal Spectrum Identifier string (e.g. mzspec:PXD004732:01650b_BC2-TUM_first_pool_53_01_01-3xHCD-1h-R2:scan:41840) resolving to an MS/MS spectrum in an online proteomics resource
- ProForma 2.0 peptidoform string (e.g. WNQLQAFWGTGK), optionally with post-translational modifications at specified amino acid positions
- Fragment mass tolerance (10 ppm in the demonstrated workflow) and fragment ion types (ion_types='aby')

## Outputs

- Annotation table of matched peak m/z values with fragment ion type, charge, and isotope annotations
- CSV file of fragment annotations (fragment_annotations.csv)

## How to apply

Load the MS/MS spectrum by its Universal Spectrum Identifier using MsmsSpectrum.from_usi from spectrum_utils.spectrum. Parse the ProForma 2.0 peptidoform string (e.g. WNQLQAFWGTGK) to obtain the amino-acid sequence and theoretical fragment ion series. Optionally apply standard preprocessing (set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity) before annotation. Call annotate_proforma with a fragment tolerance (10 ppm in the demonstrated workflow) and ion_types='aby' so observed peaks are matched to a, b, and y fragments — b/y ions are the default series. Collect matched peak m/z values together with each fragment's ion type, charge, and isotope annotation into a table, then write it out (e.g. fragment_annotations.csv).

## Related tools

- **spectrum_utils** (Provides MsmsSpectrum.from_usi for loading the spectrum by USI and annotate_proforma for matching observed peaks to a/b/y fragment ions from the ProForma 2.0 peptidoform within a 10 ppm tolerance) — https://github.com/bittremieux/spectrum_utils
- **Pyteomics** (Proteomics data processing companion cited alongside spectrum_utils for proteomics tasks in Python)
- **Python** (Execution environment for the annotation and tabulation workflow (spectrum_utils requires Python 3.6+))

## Examples

```
from spectrum_utils.spectrum import MsmsSpectrum; spectrum = MsmsSpectrum.from_usi('mzspec:PXD004732:01650b_BC2-TUM_first_pool_53_01_01-3xHCD-1h-R2:scan:41840'); spectrum.annotate_proforma('WNQLQAFWGTGK', 10e-6, 'aby')
```

## Evaluation signals

- Every row of the output table must correspond to an a, b, or y fragment match of the WNQLQAFWGTGK peptidoform within the 10 ppm tolerance used for annotate_proforma.
- Each annotated entry carries fragment ion type, charge, and isotope annotation fields; unannotated peaks should be excluded from the table.
- Matched fragment m/z values must fall within ±10 ppm of the theoretical fragment masses implied by the ProForma 2.0 peptidoform.
- The output fragment_annotations.csv parses as a well-formed table of m/z plus annotation columns.
- If preprocessing (set_mz_range 100–1400 m/z, remove_precursor_peak, filter_intensity retaining peaks ≥5% of base peak and at most the 50 most intense peaks) was applied, the annotated peaks must be a subset of the filtered peak list.

## Limitations

- spectrum_utils does not provide functionality to read spectral data files; loading spectra from local files requires external tools (USI loading is supported).
- b/y ions are the default annotation series; a-ions require explicitly requesting ion_types='aby' (or similar), so a default call will miss a-fragment matches.
- Annotation quality depends on the fragment tolerance; too large a tolerance can produce spurious matches, and the demonstrated workflow fixes this at 10 ppm.
- Only peaks matching the specified peptidoform's fragments are annotated; unassigned peaks and noise remain unannotated unless preprocessing removes them first.

## Evidence

- [methods] Annotate peaks corresponding to a, b, and y peptide fragments in the spectrum based on a ProForma 2.0 peptide string.: "Annotate peaks corresponding to a, b, and y peptide fragments in the spectrum based on a ProForma 2.0 peptide string."
- [methods] Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks.: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks."
- [readme] Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism.: "Spectrum loading from online proteomics and metabolomics data resources using the [Universal Spectrum Identifier (USI)](https://www.psidev.info/usi) mechanism."
- [discussion] spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task.: [evidence span withheld — anchor status: not_found]
- [methods] The m/z range is set to 100–1400 m/z.: [evidence span withheld — anchor status: not_found]
